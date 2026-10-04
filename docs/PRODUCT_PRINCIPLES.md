# Product Principles

These principles are release constraints. When a feature conflicts with them, the feature must be redesigned or deferred.

## P1 — Automation first

The user should not perform bookkeeping that the system can reliably perform.

Preferred flow:

1. ingest automatically;
2. classify deterministically;
3. infer cautiously;
4. ask only when necessary;
5. remember the decision.

A feature that repeatedly asks the same question is defective.

## P2 — Review exceptions, not the whole ledger

The normal user experience is an inbox of uncertain/problematic items, not a 200-row transaction table.

## P3 — Trust beats coverage

It is better to mark a transaction “unknown” than to confidently misclassify it.

Silent financial errors are worse than incomplete automation.

## P4 — User truth outranks AI

Priority:

1. transaction-specific user override;
2. user-authored rule;
3. confirmed merchant history;
4. deterministic system rule;
5. provider metadata;
6. AI;
7. unknown.

AI may suggest, but may not silently override higher-priority evidence.

## P5 — Persistent rules live in the product

Rules must survive:

- new chats;
- model changes;
- provider changes;
- device changes.

Model memory is not the source of truth.

## P6 — Spending and money movement are different

The product must never treat card payments, transfers, balance carryovers, refunds or rewards as ordinary purchases simply because they appear on a statement.

## P7 — Economic owner is not necessarily card holder

Always support:

- payment actor;
- economic owner.

This enables correct household vs personal analysis.

## P8 — Necessity and reducibility are separate

A discretionary purchase on an existing installment can be discretionary in nature but non-reducible this month.

Do not collapse these concepts.

## P9 — Every AI write is inspectable

Any ChatGPT-initiated mutation must have:

- actor;
- timestamp;
- affected entity;
- before state;
- after state;
- originating tool;
- reason/request identifier where feasible.

## P10 — Reversible by default

If a mutation can reasonably be undone, it should be.

“Undo the last thing you changed” is a required product behavior, not a convenience.

## P11 — No general-purpose AI database access

AI receives domain tools, not raw SQL.

The API should express business intent: classify, split, assign, import, reconcile, undo.

## P12 — Mobile first

The app must be comfortable on a phone.

Desktop power features may exist later, but core monthly use must not require a desktop.

## P13 — Progressive disclosure

The default transaction view should show only what helps a decision.

Advanced financial/source/parser metadata belongs behind details.

## P14 — Minimal guilt language

Use analytic labels:

- Required;
- Necessary / flexible;
- Discretionary;
- Unknown.

Avoid moral labels such as “waste”, “bad spending”, or “unnecessary” unless the user explicitly creates such a category.

## P15 — Explainability over magic

When the system classifies automatically, the user should be able to learn why:

> Rule: Özkuruşlar → Groceries → Required

or:

> AI suggestion, 78% confidence

without exposing implementation noise in the default UI.

## P16 — Source provenance matters

Every transaction should retain enough provenance to answer:

- where did this come from?
- which parser/version processed it?
- was it e-mail, statement, manual or ChatGPT?
- has it been reconciled?

## P17 — Idempotency is mandatory

Re-importing the same statement or reprocessing the same e-mail must not duplicate financial reality.

## P18 — The app is useful without AI

Core accounting semantics, parser logic, reconciliation and user rules must work without an LLM.

AI is an accelerator and interface, not the accounting engine.

## P19 — Provider independence

ChatGPT is the primary conversational interface, but business logic must not depend on a single model provider.

## P20 — Manual effort is a product metric

Track:

- review decisions per 100 transactions;
- monthly cleanup minutes;
- correction rate;
- unresolved items.

If these increase, the product is regressing even if feature count increases.
