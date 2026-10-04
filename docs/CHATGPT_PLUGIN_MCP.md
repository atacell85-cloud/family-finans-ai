# ChatGPT Plugin / MCP Integration

## 1. Objective

ChatGPT should be a first-class operator of the finance system.

The desired experience:

- user attaches a statement and asks to import it;
- user asks questions about current finance data;
- user adds cash/manual activity conversationally;
- user corrects classification/owner/category by conversation;
- user creates persistent rules;
- user asks for explanations and trend analysis;
- all writes are validated, scoped, audited and reversible when practical.

ChatGPT should not need to log in through the PWA UI or automate browser clicks.

---

## 2. Architectural rule

The integration stack is:

```text
ChatGPT
  ↓
Plugin / MCP tool adapter
  ↓
Finance API / domain commands
  ↓
Postgres + audit + rules + parsers
```

The MCP layer is an adapter, not the source of business logic.

The PWA calls the same domain layer.

---

## 3. Authentication and household scope

Never trust a household ID supplied by the model.

The tool server must derive authorization from authenticated user/session context.

A tool call may specify:
- transaction ID;
- period;
- filter;
- file reference;

but access control must independently verify that every referenced entity belongs to the authenticated household.

---

## 4. Tool design principles

Tools should represent user intentions.

Good:
- add_transaction
- classify_transaction
- split_transaction
- import_statement
- reconcile_statement
- create_rule

Bad:
- execute_sql
- patch_table
- update_any_field
- run_database_query

Schemas should be narrow enough that invalid model output is rejected before domain mutation.

---

## 5. Proposed tool catalogue

### 5.1 Read tools

#### get_dashboard
Input:
- period;
- optional view: economic | cash_debt.

Output:
- real spending;
- necessity breakdown;
- owner breakdown;
- top drivers;
- pending count;
- debt/installment summary.

#### get_month_summary
Input:
- year/month;
- comparison period optional;
- filters optional.

#### search_transactions
Input:
- period/date range;
- merchant;
- category;
- person/owner;
- payment instrument;
- amount range;
- necessity;
- type;
- tag;
- review state;
- text query.

Output should be paginated and include stable transaction IDs.

#### get_transaction
Returns business fields, source summary, reconciliation state and relevant audit/rule provenance.

#### get_pending_items
Returns actionable review items with reason and recommended decision schema.

#### get_debt_summary
Returns statement/debt/installment burden without conflating economic spending.

#### get_installment_schedule
Returns future known obligations.

#### list_rules
Filterable by merchant/type/status.

#### get_audit_history
Returns change history for an entity/change set.

---

## 6. Write tools

### add_transaction

Use for:
- cash;
- manual transfer;
- manual correction not represented elsewhere.

Required:
- amount;
- currency;
- date;
- type/direction;
- payment method/account where known.

Optional:
- merchant;
- category;
- necessity;
- owner;
- actor;
- note;
- tags.

Backend validation must decide if enough information exists to commit or create pending state.

### update_transaction

Allow only whitelisted business fields.

It must not edit:
- authenticated household scope;
- immutable source IDs;
- audit history;
- statement source evidence.

### classify_transaction

Focused classification mutation:
- category;
- subcategory;
- necessity;
- reducible_now;
- owner.

Prefer this over a generic update for AI use.

### assign_owner

Sets economic owner and optionally payment actor only when tool schema explicitly permits both.

### split_transaction

Input:
- parent transaction;
- split children with amounts and classifications.

Invariant:
sum(split amounts) = parent amount at currency precision.

### bulk_update_transactions

High-impact.

Requirements:
- search/filter result snapshot;
- preview token/change-set ID;
- affected count and total amount;
- confirmation policy.

Do not let model send an unconstrained “update everything”.

### create_rule / update_rule

Must distinguish:
- exact merchant;
- alias;
- contextual scope;
- fields set;
- whether historical application is requested.

### resolve_pending_item

Accepts only resolution fields required by the pending reason.

### undo_change

Targets a change-set ID or a safe “last eligible change” resolver.

Backend validates conflicts before reversal.

---

## 7. Import tools

### import_statement

Goal:
accept a supported file reference or uploaded file handoff and create a draft import.

Output:
- import_job_id;
- detected bank;
- period;
- parse status.

Do not commit immediately by default if reconciliation/validation is incomplete.

### preview_statement_import

Output:
- statement summary;
- arithmetic status;
- rows;
- confirmed matches;
- probable matches;
- new events;
- fees/interest/refunds;
- duplicate candidates;
- pending count;
- warnings.

### commit_statement_import

Requirements:
- import job is in committable state;
- arithmetic invariant passed or explicit reviewed adjustment exists;
- idempotency key checked;
- audit/change set created.

### reconcile_statement

May be part of preview/commit internally, but expose only if it maps to a clear user goal.

---

## 8. File handling

The implementation must perform a capability spike against the current ChatGPT/plugin environment before assuming a particular file transport mechanism.

Desired behavior:

1. user attaches statement;
2. ChatGPT receives a supported file reference;
3. tool receives the file or fetchable protected reference;
4. import service processes it.

If the active platform cannot pass the file directly:
- use a secure upload endpoint or supported file handoff;
- keep the same import API contract internally;
- do not redesign the ledger around platform-specific file mechanics.

---

## 9. Permission levels

Suggested operation classes:

### Low-risk read
- summaries;
- searches;
- transaction detail;
- rules;
- pending items.

### Low-risk write
- add one manual transaction;
- classify one transaction;
- resolve one pending field.

### Medium-impact write
- create merchant rule;
- split transaction;
- historical reclassification;
- commit statement import.

### High-impact
- large bulk update;
- delete/reverse import;
- delete account/data;
- overwrite many user-confirmed values.

High-impact operations should require explicit preview/confirmation or remain UI-only.

---

## 10. Audit requirements

Every AI write records:

- actor_type = chatgpt;
- authenticated user;
- tool name;
- request/correlation ID;
- timestamp;
- entity IDs;
- before state;
- after state;
- change-set ID;
- optional user instruction summary;
- reversible flag.

Do not store full chat content by default.

A concise reason is enough.

---

## 11. Undo behavior

“Undo what you just did” should map to a change set, not guess at database diffs.

If later edits overlap:
- do not blindly revert;
- report conflict;
- show what can be safely undone.

---

## 12. Natural-language examples

### Add
> “I paid 850 TL cash at the greengrocer today. Household.”

Expected:
- add_transaction.

### Correct
> “The 4,393 TL marketplace purchase was a shelf. Make it Home / necessary-flexible / household.”

Expected:
- search_transactions;
- get_transaction if ambiguous;
- classify/update.

### Rule
> “From now on, classify this grocery chain as Groceries / Required / Household.”

Expected:
- create_rule.

### Bulk
> “Make restaurants during my September trip Travel / Dining / Discretionary.”

Expected:
- search/preview;
- bulk update;
- change set.

### Import
> “Add this credit-card statement.”

Expected:
- import;
- preview;
- commit under permission policy.

### Analysis
> “Why did spending rise this month?”

Expected:
- get summary/drivers;
- no database mutation.

---

## 13. MCP contract tests

For each tool:

- valid call;
- invalid enum;
- inaccessible transaction;
- duplicate request_id;
- nonexistent ID;
- cross-household attempt;
- audit creation;
- undo path;
- rate limit/error behavior.

Conversation-level tests:

1. exact transaction correction;
2. ambiguous transaction asks before write;
3. merchant-wide request previews scope;
4. card payment never reclassified as purchase by conversational instruction unless explicitly changing type with protected workflow;
5. user rule survives future AI classifications.

---

## 14. Provider independence

Internal command schemas should not use ChatGPT-specific message structures.

Potential adapters:
- ChatGPT MCP/plugin;
- future Claude integration;
- internal CLI;
- admin/test harness.

The domain API remains constant.

---

## 15. Integration acceptance criteria

- ChatGPT can read current data without browser automation;
- ChatGPT can add a transaction;
- ChatGPT can classify/assign/split through bounded tools;
- ChatGPT can create persistent rules;
- ChatGPT can initiate the tested statement import workflow;
- every AI write is auditable;
- unsafe bulk/destructive writes are gated;
- no tool can escape household authorization;
- no tool exposes raw SQL;
- the system remains usable if MCP/ChatGPT is temporarily unavailable.
