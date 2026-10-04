# Test Strategy

## 1. Testing philosophy

Financial correctness and user trust matter more than nominal feature completion.

Tests should focus on:

- accounting/semantic invariants;
- parser accuracy;
- idempotency;
- reconciliation;
- rule precedence;
- authorization;
- AI write safety;
- UX critical paths.

---

## 2. Test layers

### Unit
Pure domain logic:
- money arithmetic;
- transaction typing;
- rule precedence;
- merchant normalization;
- installment calculations;
- match scoring;
- split validation.

### Integration
- Postgres/RLS;
- import persistence;
- audit;
- undo;
- API/service boundaries;
- storage access;
- worker retry.

### Golden parser tests
Synthetic/anonymized real-format statements.

### Contract tests
MCP/plugin tool schemas and domain mapping.

### E2E
Mobile-first user flows via Playwright or equivalent.

### Security
Authorization and data-leak scenarios.

---

## 3. Money handling

Never use binary floating point for currency arithmetic.

Use:
- integer minor units where practical; or
- database decimal/numeric with controlled precision.

Tests:
- rounding;
- negative/credit amounts;
- foreign currency;
- split sum equality;
- installment rounding final payment.

---

## 4. Statement parser golden tests

For each bank/template fixture assert:

- detected bank;
- period;
- account/card identity;
- row count;
- every row amount/date/type;
- previous balance;
- payments;
- period activity;
- interest/fees/taxes;
- closing balance;
- installment metadata;
- refunds/rewards;
- card-level totals.

Any parser change must rerun all prior fixtures.

---

## 5. Statement arithmetic tests

The parser must validate issuer arithmetic.

Test:
- exact pass;
- one missing row;
- wrong sign;
- duplicate row;
- malformed interest line;
- rounding boundary;
- refund;
- partial payment;
- carryover.

A mismatch must produce validation failure/review, not silent success.

---

## 6. Idempotency tests

### E-mail
Same message processed 1, 2, 10 times → one source event.

### Statement
Same file/import repeated → no new economic transactions.

### ChatGPT/manual retry
Same request ID retried after timeout → one mutation.

### Worker
Crash after source-event write, before transaction commit → retry reaches correct final state.

---

## 7. Reconciliation tests

Scenarios:

- exact e-mail ↔ statement match;
- merchant formatting differs;
- transaction date vs posting date differs;
- same amount twice same day;
- same merchant same amount same day but both real;
- wrong card;
- FX charge;
- missing e-mail;
- e-mail exists but statement never posts;
- reversed transaction;
- refund in later statement.

Assert no silent duplicate deletion.

---

## 8. Credit-card tests

Scenarios:

- simple purchase;
- 3-installment purchase;
- irregular final installment;
- refund one installment;
- full refund;
- card payment;
- carryover;
- interest;
- fees;
- taxes;
- minimum payment;
- supplementary card.

Validate economic-spending view separately from debt/cash view.

---

## 9. Rule-engine tests

- explicit transaction override wins;
- explicit user rule wins over AI;
- exact merchant alias;
- conflicting rules;
- contextual rule;
- historical reclassification;
- generic marketplace does not receive broad rule without explicit approval;
- disabling a rule;
- rule audit.

---

## 10. AI classification tests

Use fixed fixtures and mocked provider responses for deterministic CI.

Test:
- high confidence accepted where policy permits;
- medium confidence pending;
- low confidence unknown;
- malformed model response rejected;
- model suggests invalid enum rejected;
- model tries to override user rule rejected.

Separate model-quality evaluation from application correctness tests.

---

## 11. MCP tool contract tests

For every write tool:

- valid mutation;
- unauthorized entity;
- malformed fields;
- duplicate request ID;
- audit created;
- undo works where supported.

For search:
- pagination;
- household isolation;
- natural-language translation layer if present.

For bulk update:
- preview required;
- stale preview invalidation;
- scope count enforced.

---

## 12. Security tests

- anonymous read denied;
- anonymous write denied;
- user A cannot read household B;
- user A cannot mutate household B;
- signed file URL access control;
- raw storage not public;
- service key absent from client bundle;
- prompt injection in imported text cannot invoke tools;
- oversized file rejected;
- unsupported MIME rejected.

---

## 13. UX E2E tests

Critical flows:

1. onboarding with first statement;
2. quick cash add;
3. review unknown merchant;
4. create future rule from correction;
5. statement import preview;
6. duplicate resolution;
7. transaction split;
8. debt/installment view;
9. undo AI change;
10. export.

Test at mobile viewport first.

---

## 14. Regression fixture policy

Every production parser bug should result in:
- anonymized/synthetic regression fixture;
- failing test;
- fix;
- passing test.

Never fix a parser bug without locking the case.

---

## 15. Performance tests

Household scale is modest, but test:

- 5 years × several thousand transactions;
- monthly dashboard;
- natural-language-backed search filters;
- statement with hundreds of rows;
- bulk rule application.

Target interactive reads should remain comfortably sub-second to low-second depending query.

---

## 16. Backup/restore test

At least periodically:

1. backup database;
2. restore to isolated environment;
3. verify counts/invariants;
4. verify audit links;
5. verify no production secret is exposed to test environment.

---

## 17. Release gates

No release if:
- statement arithmetic fails silently;
- duplicate import is possible;
- card payment counts as spending;
- user rule can be overwritten by AI;
- RLS regression exists;
- AI write lacks audit;
- undo cannot identify a change set;
- critical mobile flow broken.

---

## 18. Test data policy

Use:
- synthetic names;
- synthetic cards;
- synthetic amounts;
- anonymized descriptions.

Do not commit real household statements or bank messages.
