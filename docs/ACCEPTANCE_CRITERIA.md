# Acceptance Criteria

## A. Product / UX

- [ ] Home screen shows real spending, necessity split and pending count before secondary details.
- [ ] Routine monthly use does not require transaction-by-transaction review.
- [ ] Quick manual entry can be completed in seconds.
- [ ] Pending item clearly states why user input is needed.
- [ ] Common correction is ≤ 2 taps or one natural-language instruction.
- [ ] Rule creation is offered contextually, not required through a complex settings screen.
- [ ] Mobile UX works on current iPhone Safari and Android Chrome.
- [ ] Language is analytical, not judgmental.

## B. Transaction semantics

- [ ] Purchase and financial movement are distinct.
- [ ] Card payment does not increase economic spending.
- [ ] Owned-account transfer does not increase household spending.
- [ ] Refund does not count as positive new expense.
- [ ] Carryover does not count as current-month spending.
- [ ] Interest/fee/tax is visible separately.
- [ ] Payment actor and economic owner are distinct fields.
- [ ] Necessity and reducible-now are distinct fields.
- [ ] Splits sum exactly to parent transaction.

## C. Installments

- [ ] Original purchase can be represented economically.
- [ ] Statement installment occurrence is linked to plan where known.
- [ ] Remaining installments/count are visible.
- [ ] Future obligation is shown separately from new spending.
- [ ] Irregular/final installment rounding is supported.
- [ ] Refund/cancellation does not create negative remaining count.

## D. Statement import

- [ ] Bank/template detection works for supported fixture.
- [ ] Statement period parsed.
- [ ] Card/sub-card identity parsed where available.
- [ ] Previous balance parsed.
- [ ] Payments parsed.
- [ ] Period activity parsed.
- [ ] Interest/fees/taxes parsed.
- [ ] Closing balance parsed.
- [ ] Statement arithmetic validated.
- [ ] Failure blocks silent commit.
- [ ] Repeat import is idempotent.
- [ ] Import preview summarizes matches/new/conflicts rather than requiring every row review.

## E. Reconciliation

- [ ] E-mail event can reconcile to statement row.
- [ ] Reconciled purchase counted once.
- [ ] Exact source duplicate auto-detected.
- [ ] Probable duplicate requires review.
- [ ] User classifications survive reconciliation.
- [ ] Statement-only charges can be added.
- [ ] Unmatched provisional items remain visible.

## F. Rules

- [ ] Transaction-specific override has highest priority.
- [ ] User rule outranks AI.
- [ ] Merchant alias resolution works.
- [ ] Ambiguous broad merchants do not get automatic global rules.
- [ ] Rule conflict produces review state.
- [ ] Rule history/audit is available.
- [ ] Historical application can be previewed.

## G. ChatGPT / MCP

- [ ] ChatGPT can read monthly summary.
- [ ] ChatGPT can search transactions.
- [ ] ChatGPT can add one transaction.
- [ ] ChatGPT can classify one transaction.
- [ ] ChatGPT can assign economic owner.
- [ ] ChatGPT can split transaction.
- [ ] ChatGPT can create/update rule.
- [ ] ChatGPT can access pending items.
- [ ] ChatGPT can initiate supported statement import workflow.
- [ ] ChatGPT can preview import result.
- [ ] ChatGPT can commit import under permission policy.
- [ ] ChatGPT can undo supported prior write.
- [ ] No raw SQL/general patch tool exists.
- [ ] Every AI write creates audit record.
- [ ] Cross-household tool access denied.

## H. Security/privacy

- [ ] RLS enabled/tested.
- [ ] Anonymous financial reads denied.
- [ ] Anonymous financial writes denied.
- [ ] Cross-household reads/writes denied.
- [ ] Statement storage private.
- [ ] Short-lived signed URLs.
- [ ] CVV/full banking password never stored.
- [ ] Secrets absent from Git/client bundle.
- [ ] Minimal mail scopes.
- [ ] Sensitive raw content excluded from ordinary logs.
- [ ] Prompt injection in imported content treated as untrusted data.
- [ ] Export/delete path documented.

## I. Audit/undo

- [ ] Mutation actor stored.
- [ ] Timestamp stored.
- [ ] Before/after state stored where appropriate.
- [ ] Change-set ID available for grouped actions.
- [ ] Supported mutations can be reversed safely.
- [ ] Conflicting later edit prevents blind undo.

## J. Reliability

- [ ] Currency arithmetic avoids floating-point error.
- [ ] Parser golden tests in CI.
- [ ] Idempotency tests pass.
- [ ] Reconciliation tests pass.
- [ ] Backup procedure exists.
- [ ] Restore procedure tested before production dependence.
- [ ] Failed import remains diagnosable.

## K. Success metrics

Target mature behavior:
- [ ] < 10 minutes monthly cleanup.
- [ ] < 5 manual decisions per 100 routine transactions.
- [ ] > 99% supported-statement row reconciliation where source data permits.
- [ ] effectively zero silent duplicate expenses.
- [ ] 100% AI writes audit logged.
