# Roadmap

The roadmap is ordered to minimize wasted work. Each phase has an exit gate. Do not begin broad automation until the previous financial and UX assumptions are proven.

---

## Phase 0 — Planning and UX lock

### Deliverables
- master plan;
- product principles;
- user journeys;
- low-fidelity mobile screens;
- dashboard information hierarchy;
- pending-review flow;
- statement-import flow;
- debt/installment flow.

### Exit gate
- UX review complete;
- second-model architecture review complete;
- open questions triaged;
- v0.2 plan approved.

---

## Phase 1 — Repository / platform foundation

### Deliverables
- monorepo scaffold;
- Next.js PWA shell;
- Supabase project;
- local/staging/prod config pattern;
- migrations;
- auth;
- RLS baseline;
- CI;
- lint/typecheck/test;
- secret scanning.

### Exit gate
- secure login;
- household-isolated sample row;
- CI green;
- no secrets in client/repo.

---

## Phase 2 — Core ledger vertical slice

### Deliverables
- household/person;
- accounts/payment instruments;
- merchants/aliases;
- transactions;
- categories;
- necessity/reducibility;
- owner/payment actor;
- audit/change sets;
- quick manual entry;
- transaction list/detail.

### Exit gate
- cash expense can be added/edited;
- audit written;
- undo basic mutation;
- economic vs financial type distinction passes unit tests.

---

## Phase 3 — First statement parser

### Deliverables
- one bank adapter;
- import jobs;
- private file upload;
- parser;
- statement summary;
- statement rows;
- arithmetic validation;
- import preview;
- golden tests.

### Exit gate
- representative statement fixture parses exactly;
- totals reconcile;
- repeat import idempotent;
- no raw sensitive fixture in Git.

---

## Phase 4 — Credit-card semantics

### Deliverables
- card hierarchy;
- supplementary card/payment actor;
- card payment;
- carryover;
- interest/fees/taxes;
- refunds;
- installment plans/occurrences;
- remaining obligation.

### Exit gate
- economic-spending and debt views both correct;
- future installment schedule correct.

---

## Phase 5 — Reconciliation engine

### Deliverables
- source-event model;
- transaction linking;
- match scoring;
- duplicate candidates;
- conflict review;
- posted/provisional state.

### Exit gate
- same purchase via two sources counted once;
- ambiguous duplicates never auto-deleted;
- user classification survives reconciliation.

---

## Phase 6 — Rules and pending inbox

### Deliverables
- rule engine;
- precedence;
- merchant rules;
- rule-from-correction UX;
- pending review reasons;
- confidence/provenance UI.

### Exit gate
- known merchants require near-zero repeated review;
- user rule override protection tested.

---

## Phase 7 — Mobile dashboard/PWA

### Deliverables
- home;
- activity;
- add/import;
- insights;
- pending inbox;
- responsive charts;
- installability.

### Exit gate
- current iPhone Safari;
- Android Chrome;
- critical flows usable one-handed;
- dashboard answers “what happened?” immediately.

---

## Phase 8 — ChatGPT capability spike

### Deliverables
Prove current platform supports required workflow:

1. authenticated read;
2. add one manual transaction;
3. classify one transaction;
4. create one rule;
5. statement file handoff or supported equivalent;
6. audit every write.

### Exit gate
Document exact supported capabilities and limitations.

Do not build the full MCP surface before this passes.

---

## Phase 9 — Full ChatGPT/MCP tools

### Deliverables
- read catalogue;
- bounded write catalogue;
- import preview/commit;
- bulk preview/change set;
- undo;
- contract tests;
- permission/risk policy.

### Exit gate
Representative natural-language tasks succeed safely.

---

## Phase 10 — Gmail ingestion

### Deliverables
- narrow OAuth scope;
- sender allowlist;
- first bank e-mail parser;
- polling/webhook;
- idempotency;
- provisional transaction;
- ingestion error queue.

### Exit gate
Routine known merchant purchase requires zero user work.

---

## Phase 11 — AI fallback

### Deliverables
- provider abstraction;
- unknown transaction classifier;
- confidence policy;
- pending suggestion UI;
- quality evaluation set.

### Exit gate
AI reduces review load without increasing correction rate beyond agreed threshold.

---

## Phase 12 — Advanced insights

### Deliverables
- month delta decomposition;
- normal month;
- event/travel exclusion;
- household vs personal;
- reducible spend;
- category anomaly;
- installment forecast.

### Exit gate
Explanations are fact-based and reconcile to transaction totals.

---

## Phase 13 — Additional banks/sources

Each adapter:
- specification;
- fixtures;
- parser;
- arithmetic rules;
- reconciliation tests;
- release flag.

No bank adapter ships without golden tests.

---

## MVP milestone

MVP is reached after Phases 0–9 if statement import and ChatGPT workflows are stable. Gmail and AI fallback are important automation layers but should not block validating the core product.

---

## Stop rules

Pause and redesign if:
- manual review load is rising;
- statement arithmetic cannot be made deterministic for first bank;
- ChatGPT write workflow requires fragile browser automation;
- data model forces card payment/installment double counting;
- RLS/security cannot be confidently verified.

---

## Time-saving principle

Build one end-to-end vertical slice before broad feature work.

Avoid:
- supporting five banks simultaneously;
- building a sophisticated category editor;
- custom chart framework;
- complex offline sync;
- public SaaS features.

The user wants a reliable personal system, not a startup platform.
