# Master Plan

**Project:** Family Finans AI  
**Status:** Planning baseline v0.1  
**Primary objective:** Build a mobile-first household finance system whose default mode is automation, not bookkeeping.

---

## 1. Product thesis

Most personal-finance tools make the user behave like an accountant: enter transactions, clean categories, reconcile cards, repair duplicates, and maintain rules. This project reverses that relationship.

The system should:

1. ingest financial activity automatically where practical;
2. treat bank statements as authoritative reconciliation evidence, not just flat lists of expenses;
3. keep deterministic user rules outside the AI model;
4. ask the user only when confidence is insufficient;
5. expose the same domain operations to both the PWA and ChatGPT;
6. preserve a complete audit trail and reversible changes;
7. explain household spending in human terms.

The product is successful when the user can forget about it for most of the month and still receive a trustworthy answer to questions such as:

- How much did we actually spend this month?
- How much of that was required versus discretionary?
- Who used the card, and who/what did the expense belong to?
- Why did this month increase?
- What recurring installment burden remains?
- What was our normal monthly cost excluding travel or one-offs?
- Which transactions still need my judgment?

The target operating metric is **under 10 minutes of manual cleanup per month** after the system has learned common merchants.

---

## 2. Non-negotiable product principles

The detailed principles are in [PRODUCT_PRINCIPLES.md](PRODUCT_PRINCIPLES.md). The following are architectural constraints, not suggestions:

- The UI must optimize for review, not data entry.
- The system must distinguish **economic spending** from **financial movement**.
- User-confirmed data outranks AI inference.
- Persistent rules live in the application database, not in model memory.
- A transaction may have a payment instrument user and a separate economic owner.
- AI cannot have unrestricted SQL access.
- Every AI write must be attributable, auditable, and reversible when practical.
- Statement imports must be idempotent.
- Duplicate detection must be evidence-based, not simply “same date + same amount”.
- Card payments must never be counted as new spending.
- Installments must support both purchase-time economic analysis and payment-time cash/debt analysis.
- Sensitive bank credentials, CVV and full PAN must never be stored.

---

## 3. Product scope

### 3.1 MVP scope

The MVP must support:

- one household;
- multiple people within the household;
- bank/card accounts and cash accounts;
- transaction ingestion from:
  - PDF statements;
  - CSV/XLSX where available;
  - bank notification e-mails;
  - manual/quick entry;
  - ChatGPT tools;
- transaction types:
  - purchase;
  - installment charge;
  - refund;
  - card payment;
  - interest;
  - fee;
  - tax;
  - cash advance;
  - reward;
  - discount;
  - balance carryover;
  - transfer;
  - income;
  - adjustment;
- merchant normalization and aliases;
- category and subcategory;
- necessity classification:
  - required;
  - necessary/flexible;
  - discretionary;
  - unknown;
- independent “reducible now?” dimension;
- person who used the payment method;
- economic owner:
  - household;
  - person;
  - optionally child/dependent later;
- installment schedule and remaining obligation;
- statement reconciliation;
- duplicate detection;
- pending-review inbox;
- deterministic merchant rules;
- dashboard and trend charts;
- audit log;
- undo for supported mutations;
- ChatGPT read/write/import tools;
- mobile-first PWA.

### 3.2 Explicitly out of scope for MVP

Do not add these until the core system is stable:

- brokerage / investment portfolio tracking;
- crypto prices;
- bill payment;
- money transfers;
- bank credential scraping;
- PSD2/open-banking aggregation unless a later provider makes it clearly worthwhile;
- credit scoring;
- tax filing;
- complex double-entry accounting UI;
- financial advice engine;
- public multi-tenant SaaS onboarding;
- family-member social features;
- gamification.

---

## 4. Core mental model

### 4.1 Financial event is not always an expense

A bank statement is a mixture of different economic meanings.

Examples:

- a card purchase is spending;
- a card payment is debt settlement;
- a refund reverses or reduces prior spending;
- interest is a financing cost;
- balance carryover is not new spending;
- a reward can offset cash cost without being a normal income source;
- an installment charge may relate to a purchase made months earlier.

The system therefore maintains two views:

#### Economic view
“What did the household consume or buy?”

#### Cash/debt view
“What money or liability moved this period?”

Both are valid. They must not be merged into one number.

### 4.2 Payment actor vs economic owner

Example:

A household member uses their card to buy groceries.

- payment actor: the card holder/user;
- economic owner: household;
- category: groceries;
- necessity: required.

This distinction is mandatory because “whose card was charged?” and “whose personal spending was this?” are different questions.

### 4.3 Necessity vs current reducibility

Example:

A discretionary motorcycle accessory bought on a 6-month installment plan.

- original spending nature: discretionary;
- current installment due: not reducible now;
- remaining obligation: yes.

The product must not label all “unavoidable this month” items as required spending.

---

## 5. Primary user experience

The definitive UX is specified in [UX_SPEC.md](UX_SPEC.md) and [USER_JOURNEYS.md](USER_JOURNEYS.md).

The home screen should answer “what happened?” before offering controls.

The user should normally see:

- real spending this month;
- required / flexible / discretionary / unknown split;
- household vs personal split;
- key category movement;
- number of pending items;
- debt/installment snapshot;
- month-over-month change.

The system should not present a giant transaction grid as the default screen.

### 5.1 Pending-review philosophy

The system only asks when it lacks sufficient confidence or detects inconsistency.

An item enters the review inbox when, for example:

- merchant is unknown;
- category confidence is low;
- necessity is uncertain;
- owner is unclear;
- duplicate probability is material;
- statement reconciliation failed;
- a parsed amount/date/card identity is uncertain;
- a rule conflict exists.

A known merchant with a user-confirmed rule should not repeatedly require review.

---

## 6. Ingestion strategy

Detailed design: [DATA_INGESTION.md](DATA_INGESTION.md).

### 6.1 Bank e-mail notifications

Use bank e-mails as near-real-time provisional events.

Flow:

Bank → mailbox → ingestion worker → normalized candidate → rule engine → transaction/review inbox.

E-mail ingestion should use:

- sender allowlist;
- bank-specific parsers where possible;
- provider message ID for idempotency;
- minimal retention of raw content;
- retry queue;
- parser confidence.

### 6.2 Statements

Statements are authoritative reconciliation sources.

They serve two jobs:

1. initial historical import;
2. monthly confirmation/reconciliation of provisional activity.

The user should not need to inspect every statement line. The import result should summarize:

- how many rows were found;
- how many matched existing events;
- how many new events were found;
- how many probable duplicates exist;
- how many financial costs/fees appeared;
- how many items require review.

### 6.3 ChatGPT

ChatGPT is a first-class operator, not merely a reporting chatbot.

Examples:

- “Add 850 TL cash groceries paid by Büşra.”
- “The 4,393 TL Amazon transaction was a shelf for the children’s room; classify it as household / home / necessary.”
- “Mark all Toyzz Shop spending as children / discretionary.”
- “Import this statement.”
- “Show me why September spending rose.”
- “Undo the last classification change.”

### 6.4 Manual UI

Manual forms exist for fallback and correction, but they should be progressively disclosed and short.

---

## 7. Statement parsing strategy

### 7.1 Adapter architecture

Use bank-specific adapters:

- VakifBankAdapter;
- GarantiAdapter;
- IsBankAdapter;
- AkbankAdapter;
- GenericStatementAdapter.

Each adapter should expose the same normalized contract.

### 7.2 Parsing pipeline

Preferred order:

1. native PDF text extraction;
2. bank-specific deterministic parser;
3. table/layout extraction;
4. OCR/vision fallback;
5. structured AI extraction only when deterministic parsing is insufficient.

AI output must carry confidence and source provenance. It must never silently overwrite a deterministic or user-confirmed value.

### 7.3 Golden fixtures

Each supported bank requires synthetic or irreversibly anonymized golden statement fixtures.

A fixture should assert:

- statement period;
- previous balance;
- payments;
- period transactions;
- interest/fees/taxes;
- statement balance;
- card-level subtotals;
- installment rows and remaining counts;
- refunds/rewards;
- row count and normalized event types.

---

## 8. Reconciliation

Detailed design: [STATEMENT_RECONCILIATION.md](STATEMENT_RECONCILIATION.md).

The central rule: **a statement import must not create a second expense when the same purchase already arrived by e-mail.**

Matching signals include:

- masked card/account;
- merchant normalized identity;
- amount;
- currency;
- transaction date/time;
- posting date;
- source message ID/reference;
- statement reference;
- direction;
- transaction type.

Reconciliation outcomes:

- confirmed match;
- probable match requiring review;
- unmatched provisional;
- new statement-only event;
- duplicate candidate;
- conflict.

User edits made before reconciliation must be preserved. Reconciliation enriches source truth; it must not erase classification decisions.

---

## 9. Classification and rule engine

Detailed design: [CLASSIFICATION_RULES.md](CLASSIFICATION_RULES.md).

Decision precedence:

1. transaction-specific user override;
2. explicit user rule;
3. previously confirmed merchant mapping;
4. deterministic system rule;
5. bank/provider metadata;
6. AI inference;
7. unknown.

A rule may set:

- canonical merchant;
- category;
- subcategory;
- necessity;
- economic owner;
- payment actor default;
- reducible-now default;
- tags;
- travel/event scope.

Rules must be inspectable and reversible.

AI should not be called for every transaction. It is a fallback, not the primary classification engine.

---

## 10. Credit-card model

Detailed design: [CREDIT_CARD_MODEL.md](CREDIT_CARD_MODEL.md).

The product must represent:

- purchase date;
- posted date;
- statement period;
- statement due date;
- original purchase amount;
- installment number;
- installment total count;
- current installment charge;
- remaining installment amount/count;
- refund linkage;
- payment events;
- carried balance;
- interest and fee events.

The dashboard must be able to answer both:

- “How much did we buy?”
- “How much card liability falls due?”

without double counting.

---

## 11. ChatGPT / MCP integration

Detailed design: [CHATGPT_PLUGIN_MCP.md](CHATGPT_PLUGIN_MCP.md).

The domain backend is the authority. ChatGPT interacts through bounded tools.

Initial tool families:

### Read
- get_dashboard
- get_month_summary
- search_transactions
- get_transaction
- get_debt_summary
- get_installment_schedule
- list_rules
- get_pending_items
- get_audit_history

### Write
- add_transaction
- update_transaction
- classify_transaction
- assign_owner
- split_transaction
- bulk_update_transactions
- create_rule
- update_rule
- resolve_pending_item
- undo_change

### Import/reconcile
- import_statement
- preview_statement_import
- commit_statement_import
- reconcile_statement

There must be no general-purpose SQL tool.

High-impact operations require additional confirmation or may remain UI-only.

The integration layer must be replaceable. Core business logic must not depend on one AI provider.

---

## 12. Security and privacy

Detailed design: [SECURITY_PRIVACY.md](SECURITY_PRIVACY.md).

Baseline requirements:

- Supabase/Postgres with Row Level Security from day one;
- least-privilege service accounts;
- no bank login credentials;
- no CVV;
- only masked PAN / last four digits;
- encrypted secrets in hosting secret manager;
- minimal Gmail scopes;
- sender allowlist;
- raw e-mail retention minimized;
- private storage for statements;
- signed URLs with short expiry;
- comprehensive mutation audit log;
- sensitive values redacted from application logs;
- export/delete capability;
- backup and recovery procedure.

Because the repository may be public, no real financial statements, names, card data, e-mail bodies or private tokens may be committed.

---

## 13. Proposed technical architecture

Detailed design: [ARCHITECTURE.md](ARCHITECTURE.md).

Recommended baseline:

- **Web/PWA:** Next.js + TypeScript;
- **UI:** Tailwind-based component system;
- **Database/Auth/Storage:** Supabase/Postgres;
- **Charts:** Recharts or equivalent;
- **Domain packages:** TypeScript;
- **MCP server:** Node/TypeScript using the current supported MCP SDK;
- **Hosting:** Vercel for web, compatible serverless/container host for MCP/ingestion;
- **Jobs:** Supabase scheduled functions / queue / hosted worker depending reliability needs;
- **Testing:** Vitest/Jest + Playwright + parser fixture tests.

Suggested monorepo:

```text
apps/
  web/
  mcp/
  worker/
packages/
  domain/
  database/
  parsers/
  rules/
  analytics/
  shared/
supabase/
  migrations/
tests/
  fixtures/
  golden/
docs/
```

---

## 14. Delivery phases

### Phase 0 — Product/UX lock

**Goal:** Validate the interaction model before infrastructure.

Deliverables:

- low-fidelity mobile flows;
- dashboard hierarchy;
- review inbox;
- transaction detail;
- quick add;
- statement import result;
- debt/installment view;
- rule confirmation UX.

Exit criteria:

- a new user can understand the main screen in under two minutes;
- a statement import requires review only of exceptions;
- common corrections take at most two taps or one natural-language instruction.

### Phase 1 — Domain ledger

Build:

- household;
- people;
- accounts/cards/cash;
- merchants;
- transactions;
- ownership;
- categories;
- necessity;
- audit trail;
- reversible mutations.

No AI required.

Exit criteria:

- manual transactions and edits are correct;
- card payments are excluded from spending;
- refunds and transfers are represented correctly;
- core financial invariants pass tests.

### Phase 2 — First bank statement adapter

Start with one real bank format.

Build:

- native text parser;
- normalized statement model;
- transaction typing;
- installment extraction;
- summary extraction;
- import preview;
- golden fixture tests.

Exit criteria:

- parser outputs match the known statement totals;
- statement arithmetic reconciles;
- import is idempotent.

### Phase 3 — Reconciliation engine

Build:

- source-event model;
- transaction fingerprinting;
- match scoring;
- conflict workflow;
- duplicate review;
- provisional-to-posted promotion.

Exit criteria:

- e-mail + statement does not double count;
- user classifications survive reconciliation.

### Phase 4 — Rules and pending inbox

Build:

- merchant aliases;
- deterministic rule engine;
- rule creation from correction;
- confidence model;
- pending reasons.

Exit criteria:

- repeated merchants stop generating unnecessary work;
- rule conflicts are visible and explainable.

### Phase 5 — Mobile PWA

Build:

- home dashboard;
- pending inbox;
- transactions;
- import flow;
- debt/installments;
- quick add;
- search/filter.

Exit criteria:

- iPhone Safari and Android Chrome usable;
- installable PWA;
- basic offline read and queued quick-add if feasible.

### Phase 6 — ChatGPT integration spike

Before broad implementation, verify the current ChatGPT/plugin/MCP environment supports the required file and write workflows for the user’s plan and surface.

Build a minimal end-to-end test:

1. ChatGPT reads monthly summary.
2. ChatGPT adds one cash transaction.
3. ChatGPT updates one classification.
4. ChatGPT receives/imports one test statement or an equivalent supported file reference.
5. Audit log captures every mutation.

If a platform limitation exists, preserve the same Finance API and swap only the adapter.

### Phase 7 — Full ChatGPT tool set

Add:

- transaction search;
- bulk update;
- rule management;
- import preview/commit;
- reconciliation;
- undo;
- analytical queries.

Exit criteria:

- representative natural-language tasks pass contract tests;
- destructive operations remain gated.

### Phase 8 — Gmail ingestion

Build:

- narrow search/sender allowlist;
- bank-specific e-mail parser;
- provider message ID idempotency;
- retry/error queue;
- provisional transactions.

Exit criteria:

- duplicate notification ingestion is impossible;
- failures appear in an actionable queue;
- no mailbox-wide content is unnecessarily retained.

### Phase 9 — AI fallback classification

Add AI only for unresolved cases.

AI must emit:

- proposed values;
- confidence;
- rationale/provenance metadata.

Exit criteria:

- precision target agreed and measured;
- low-confidence results remain pending;
- AI cannot override user-confirmed data.

### Phase 10 — Advanced analytics

Add:

- month-over-month drivers;
- “normal month” calculation;
- travel/one-off exclusion;
- household vs personal decomposition;
- avoidable-spend trend;
- future installment obligation;
- category anomaly detection.

### Phase 11 — Additional bank adapters

Each new bank is an isolated adapter with golden fixtures and arithmetic invariants.

---

## 15. Testing strategy

Full strategy: [TEST_STRATEGY.md](TEST_STRATEGY.md).

Critical test classes:

- domain invariant tests;
- parser golden tests;
- statement arithmetic tests;
- duplicate/reconciliation tests;
- installment tests;
- refund linkage tests;
- rule precedence tests;
- mutation authorization tests;
- audit/undo tests;
- MCP tool contract tests;
- end-to-end mobile flows;
- RLS/security tests.

A statement import must fail loudly or enter review if totals cannot be reconciled.

---

## 16. Core financial invariants

The implementation must define exact sign conventions, but conceptually:

**closing balance = previous balance + net new charges + interest/fees/taxes − payments/credits**

Each bank adapter maps its statement semantics into this invariant.

Additional invariants:

- card payment cannot increase economic spending;
- transfer between owned accounts cannot increase net household spending;
- refund cannot be counted as new spending;
- split children must sum exactly to the parent amount;
- statement import rerun must not change net totals;
- one source event cannot be committed twice;
- user-confirmed classification cannot be silently replaced by AI;
- installment remaining count cannot become negative;
- audit log must identify actor and previous/new state for mutable business fields.

---

## 17. Observability

Record non-sensitive operational metrics:

- imports attempted/succeeded/failed;
- parser type/version;
- rows found;
- rows matched;
- rows newly created;
- duplicate candidates;
- pending count;
- reconciliation rate;
- auto-classification rate;
- user correction rate;
- AI correction precision where measurable;
- average manual decisions per 100 transactions.

Do not log raw statement text or unmasked card data.

---

## 18. Success metrics

Primary:

- manual review time per month;
- pending items per 100 transactions;
- statement reconciliation rate;
- duplicate rate;
- classification correction rate;
- percentage of transaction value with confirmed category/necessity/owner.

Suggested mature targets:

- < 10 minutes manual cleanup/month;
- < 5 manual decisions per 100 routine transactions;
- > 99% statement line reconciliation where bank format is supported;
- effectively zero silent duplicate expenses;
- 100% AI writes audit logged.

---

## 19. Operational model

### Backups
- scheduled database backups;
- periodic restore test;
- user export in CSV/JSON.

### Migrations
- schema changes only through versioned migrations;
- no ad-hoc production edits.

### Environments
- local;
- staging;
- production.

Synthetic/anonymized fixtures only outside production.

### Feature flags
Use flags for:

- new bank parsers;
- AI classification;
- write-capable ChatGPT tools;
- experimental reconciliation rules.

---

## 20. Risk register

### R1 — Incorrect statement parsing
**Impact:** wrong financial totals.  
**Mitigation:** deterministic adapters, golden fixtures, arithmetic reconciliation, import preview, fail-to-review rather than silent acceptance.

### R2 — Duplicate spending
**Impact:** destroys user trust.  
**Mitigation:** source IDs, fingerprints, reconciliation engine, idempotency constraints.

### R3 — AI mutation error
**Impact:** incorrect classifications or records.  
**Mitigation:** bounded tools, audit, undo, confirmation for high-impact operations, no raw SQL.

### R4 — Platform dependency
**Impact:** ChatGPT plugin behavior changes.  
**Mitigation:** core Finance API independent of ChatGPT; adapter boundary; capability spike before deep implementation.

### R5 — Scope creep
**Impact:** project becomes another long-running platform.  
**Mitigation:** strict MVP exclusions and phase gates.

### R6 — Privacy exposure
**Impact:** sensitive household financial data leak.  
**Mitigation:** private data stores, RLS, minimal retention, no real fixtures in Git, least privilege.

### R7 — Over-automation
**Impact:** confident but wrong classification.  
**Mitigation:** confidence thresholds; deterministic/user rules outrank AI; unknown is a valid state.

### R8 — Poor UX despite correct backend
**Impact:** abandonment.  
**Mitigation:** Phase 0 UX lock before implementation; manual-touch metric is a release criterion.

---

## 21. Definition of Done for MVP

MVP is not done because the app launches. It is done when all of the following are true:

1. Mobile-first UI works on current iPhone Safari and Android Chrome.
2. One supported real bank statement format imports correctly.
3. Statement totals reconcile.
4. Repeat imports are idempotent.
5. Cash/manual entry takes seconds.
6. Household/person ownership is supported.
7. Required/flexible/discretionary/unknown classification works.
8. “Reducible now?” is separate from necessity.
9. Installments are modeled correctly.
10. Remaining installment burden is visible.
11. Card payments are not counted as spending.
12. Refunds do not inflate spending.
13. Transfers do not inflate household spending.
14. Pending inbox only surfaces uncertainty/problems.
15. Merchant rules reduce repeat work.
16. Audit log exists.
17. Undo exists for supported writes.
18. ChatGPT can read summaries.
19. ChatGPT can search transactions.
20. ChatGPT can add a transaction.
21. ChatGPT can modify classification/ownership.
22. ChatGPT can create/update a user rule.
23. ChatGPT can initiate supported statement import or the tested equivalent for the active platform.
24. Every AI write is logged.
25. User-confirmed values are protected from silent AI override.
26. Security/RLS tests pass.
27. Export works.
28. Backup/restore process is documented.
29. Representative end-to-end flows pass.
30. The user can complete a normal monthly reconciliation with minimal manual effort.

---

## 22. Decision gates

Before coding beyond the first vertical slice, resolve:

1. Is the repository private? If not, confirm no sensitive fixtures will ever be committed.
2. Which exact bank/e-mail format is first?
3. Which deployment setup minimizes operational work?
4. Does the active ChatGPT environment support the required read/write/file workflow today?
5. What operations require explicit confirmation?
6. How long, if at all, are raw statement files/e-mail bodies retained?
7. What is the exact rule for economic purchase recognition on installments?
8. What is the minimum offline requirement?
9. Which chart set is genuinely useful versus decorative?
10. What constitutes a “normal month” for analytics?

These are tracked in [OPEN_QUESTIONS.md](OPEN_QUESTIONS.md).

---

## 23. Recommended first vertical slice

Do not start with the full platform.

Build one thin, complete path:

1. mobile dashboard shell;
2. manual transaction;
3. one bank statement parser;
4. import preview;
5. commit;
6. classification;
7. pending review;
8. monthly summary;
9. audit log;
10. one ChatGPT read tool and one low-risk write tool.

This exposes UX, financial-model and integration problems before Gmail automation or advanced AI work.

---

## 24. Plan review process

Before implementation:

1. Freeze this v0.1 planning baseline.
2. Review with an independent model using [CLAUDE_REVIEW_PROMPT.md](CLAUDE_REVIEW_PROMPT.md).
3. Save the unedited review as `docs/CLAUDE_REVIEW.md`.
4. Triage each finding:
   - ACCEPT;
   - PARTIAL;
   - REJECT;
   - DEFER.
5. Record architectural changes in [DECISIONS.md](DECISIONS.md).
6. Publish Master Plan v0.2.
7. Only then begin implementation milestones.

The goal of the second-model review is not to redesign the product by taste. It is to find missing edge cases, security flaws, financial-model errors, UX friction and unnecessary scope before code makes them expensive.
