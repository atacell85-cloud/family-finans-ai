# Architecture

## 1. Architecture goals

The system must be:

- correct enough for personal financial decisions;
- mobile-first;
- inexpensive to operate;
- easy to evolve;
- independent of any single AI provider;
- safe for write-capable AI access;
- auditable;
- resilient to duplicate ingestion and parser failures.

The architecture should remain simple until reliability requirements justify added infrastructure.

---

## 2. Logical components

### 2.1 Web / PWA

Responsibilities:

- authentication;
- dashboard;
- review inbox;
- transaction search/detail/edit;
- statement upload/import preview;
- debt/installment views;
- quick add;
- rule confirmation;
- settings/export.

It should call domain APIs, not manipulate database tables directly beyond approved backend patterns.

### 2.2 Finance API / domain layer

The authority for business operations.

Responsibilities:

- validation;
- transaction lifecycle;
- ownership;
- classification;
- splits;
- reconciliation;
- installment logic;
- rule precedence;
- audit logging;
- undo;
- analytics queries.

Both UI and AI tools must converge here.

### 2.3 Ingestion worker

Responsibilities:

- e-mail polling/webhook intake;
- file intake;
- parser execution;
- normalization;
- retry;
- dead-letter/error state;
- source-event idempotency.

### 2.4 Parser packages

One adapter per bank/provider plus generic fallback.

Each parser converts source-specific content into normalized import candidates without directly mutating business records.

### 2.5 Rule engine

Pure/deterministic as far as possible.

Input:
- normalized candidate;
- merchant;
- household/user rules;
- confirmed history.

Output:
- proposed canonical merchant;
- category;
- necessity;
- owner;
- confidence/provenance.

### 2.6 Reconciliation engine

Matches source events representing the same underlying financial reality.

It must support provisional e-mail events and posted statement events.

### 2.7 Analytics engine

Computes:
- economic spending;
- cash/debt movement;
- category/person trends;
- avoidable spend;
- month deltas;
- installment obligations.

### 2.8 MCP / ChatGPT adapter

Presents bounded tools to ChatGPT.

It should be thin: authorization + schema + mapping into domain operations.

No business rules should exist only in MCP tool handlers.

---

## 3. Recommended repository layout

```text
apps/
  web/
  mcp/
  worker/
packages/
  domain/
  database/
  parsers/
    vakifbank/
    generic/
  rules/
  reconciliation/
  analytics/
  shared/
supabase/
  migrations/
  seed/
tests/
  fixtures/
  golden/
  e2e/
docs/
```

---

## 4. Data-store boundaries

### Postgres
Primary structured source of truth.

Stores:
- household/person;
- account/payment instrument;
- merchant/aliases;
- transactions;
- source events;
- statements/import jobs;
- classification/rules;
- installments;
- audit/change sets;
- pending reviews.

### Object storage
Private statement files and optional short-lived raw import artifacts.

Rules:
- private bucket;
- signed URL;
- short expiry;
- configurable retention.

### Secrets
Hosted secret manager/environment:
- OAuth client secrets;
- service keys;
- MCP secrets;
- model provider keys.

Never in database rows intended for user export or in Git.

---

## 5. Suggested database entities

This is conceptual, not final schema.

### identity
- households
- people
- users / household_memberships

### finance structure
- accounts
- payment_instruments
- merchants
- merchant_aliases
- categories

### activity
- source_events
- transactions
- transaction_splits
- transaction_links
- installment_plans
- installment_occurrences

### ingestion
- import_jobs
- statements
- statement_rows
- ingestion_errors

### classification
- rules
- classification_evidence
- pending_reviews

### operations
- change_sets
- audit_events
- undo_links

Keep source parsing separate from committed transaction state.

---

## 6. Transaction lifecycle

Suggested states:

```text
candidate
  ↓
normalized
  ↓
classified
  ↓
provisional
  ↓ statement reconciliation
posted / confirmed
```

Alternative branches:

- pending_review;
- duplicate_candidate;
- conflict;
- rejected;
- reversed.

Do not overload one status field with unrelated concepts. Posting state, review state and reconciliation state may need separate fields.

---

## 7. Source-event model

A source event is evidence from an external or manual source.

Examples:

- one bank e-mail;
- one PDF statement row;
- one CSV row;
- one manual entry;
- one ChatGPT instruction.

Multiple source events may link to one transaction.

This is essential for e-mail → statement reconciliation.

Example:

```text
email source event ─┐
                    ├── transaction: Grocery purchase
statement row ──────┘
```

The classification belongs to the transaction, not to each source event.

---

## 8. API design style

Use intent-oriented endpoints/domain commands.

Good:

- createManualTransaction
- classifyTransaction
- assignOwner
- splitTransaction
- createMerchantRule
- previewStatementImport
- commitStatementImport
- reconcileStatement
- undoChangeSet

Avoid exposing AI to:

- generic PATCH table endpoint;
- raw SQL;
- arbitrary field mutation.

The web UI may use similar service methods.

---

## 9. Authorization

### User/session
Supabase Auth or equivalent.

### Database
RLS:
- every business row linked directly or indirectly to household_id;
- session user must be a household member;
- service-role access limited to trusted backend workers.

### MCP
User-authenticated binding to household.

The MCP server must never accept a household_id supplied by the model as sufficient authorization. Resolve household scope from authenticated identity.

---

## 10. Async work

Use async jobs for:

- PDF parsing;
- OCR;
- AI fallback extraction;
- reconciliation of large imports;
- e-mail ingestion;
- analytics recomputation if expensive.

Simple manual transaction writes remain synchronous.

Start with the simplest reliable job mechanism available in the chosen platform. Do not add Kafka or heavy infrastructure.

---

## 11. Event/idempotency design

All ingestion operations should carry an idempotency key.

Examples:

- Gmail message ID;
- file content hash;
- bank transaction reference;
- statement identifier + row fingerprint;
- client-generated request ID for ChatGPT/manual writes.

Database uniqueness constraints should enforce idempotency where possible.

---

## 12. Versioning

Version:

- parser adapters;
- rule engine behavior;
- normalized import schema;
- MCP tool schemas.

Store parser version on import/source evidence so historical parser bugs can be diagnosed.

---

## 13. Analytics architecture

Prefer query/view/materialized-view strategy before introducing a separate analytics warehouse.

For household scale, Postgres can comfortably answer most analytics.

Potential derived views:

- economic_spending_view;
- cashflow_view;
- debt_obligation_view;
- monthly_spending_summary;
- ownership_summary;
- necessity_summary.

Materialize only if real performance data justifies it.

---

## 14. AI provider boundary

Define internal interfaces such as:

- classifyUnknownTransaction(candidate, context)
- extractUnknownStatementStructure(file/text)
- summarizeSpendingDelta(periodA, periodB, facts)

The provider adapter may call OpenAI, Anthropic, Google or local model.

Do not let provider response shapes leak into domain tables.

---

## 15. Deployment

Suggested minimal setup:

### Web
Vercel or equivalent.

### Database/Auth/Storage
Supabase.

### MCP
Same TypeScript monorepo, deployed separately if runtime requirements differ.

### Worker
Initially serverless scheduled function or lightweight worker.

Add durable queue only when ingestion volume/failure modes justify it.

---

## 16. Environments

### local
Synthetic fixtures and local database.

### staging
Synthetic/anonymized data only.

### production
Real household financial data.

Never copy production statements into development environments.

---

## 17. Observability

Capture:

- request ID;
- actor type;
- import job ID;
- parser/version;
- non-sensitive error code;
- timing;
- counts.

Never log:

- full card number;
- CVV;
- auth token;
- raw bank e-mail body by default;
- full statement text by default.

---

## 18. Failure philosophy

Prefer:

> stop + explain + request review

over:

> silently approximate.

Financial trust is the product.

Examples:

- statement arithmetic mismatch → import stays draft;
- ambiguous duplicate → review;
- unknown owner → unknown/pending;
- parser confidence low → no silent commit.

---

## 19. Architecture acceptance criteria

The architecture is acceptable only if:

- UI and MCP use common domain operations;
- parser code cannot bypass audit/validation to create arbitrary final transactions;
- e-mail and statement can represent the same transaction without duplication;
- user rules are independent of AI provider;
- RLS isolates household data;
- source provenance is retained;
- imports are idempotent;
- AI writes are reversible/audited;
- statement parser can fail safely;
- core product continues to function if AI provider is disabled.
