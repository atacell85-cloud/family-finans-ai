# Statement Reconciliation

## 1. Objective

Turn multiple observations of the same financial event into one trustworthy transaction.

Primary case:

> Bank e-mail arrives when purchase occurs.  
> Monthly statement later contains the posted purchase.  
> System must end with one transaction, not two.

---

## 2. Evidence model

A transaction may have multiple source events.

Example:

- source A: Gmail notification;
- source B: statement row.

The transaction contains business classification and ownership.

The source events contain evidence/provenance.

---

## 3. Matching inputs

Potential signals:

- masked card/account;
- amount;
- currency;
- merchant canonical/alias;
- transaction timestamp;
- posting date;
- provider transaction reference;
- statement row identifier;
- direction;
- type;
- installment metadata.

Signals should have weights and hard constraints.

Examples:

- different currency may be hard conflict;
- different card may strongly reduce match;
- exact external reference may be definitive;
- amount/date/merchant similarity may be probabilistic only.

---

## 4. Match outcomes

### CONFIRMED
Evidence is strong enough to merge automatically.

### PROBABLE
Likely same event but needs review.

### NO_MATCH
Create or retain separate transaction.

### CONFLICT
Evidence suggests same source lineage but fields disagree materially.

### DUPLICATE_SOURCE
Same e-mail/file row already processed.

---

## 5. Preservation rules

When a provisional transaction was already edited by the user:

- preserve category;
- preserve necessity;
- preserve owner;
- preserve user notes;
- preserve explicit merchant override.

Statement reconciliation may enrich:

- posted date;
- authoritative description;
- statement ID;
- installment details;
- bank reference;
- posting status.

It must not reset user-confirmed business meaning.

---

## 6. Statement arithmetic validation

Each bank adapter defines sign semantics.

Conceptually validate:

```text
closing_balance
≈ opening_balance
+ net_charges
+ interest
+ fees
+ taxes
- payments
- credits/refunds
```

Tolerance must be exact for currency minor units unless bank rounding/FX rules explicitly justify a small difference.

If validation fails:

- do not silently commit;
- mark validation_failed;
- show difference;
- rank suspicious rows;
- retain import draft.

---

## 7. Card-level reconciliation

Statements with multiple cards should reconcile:

- statement total;
- per-card subtotals if supplied;
- cardholder/sub-card identity.

A sub-card transaction belongs to the correct payment actor even when statement account is shared.

---

## 8. Installment reconciliation

A statement installment row should link to an installment plan where possible.

Match:

- merchant;
- plan/original purchase reference;
- installment amount;
- sequence number;
- original date;
- card.

If no plan exists, create a provisional plan candidate rather than pretending the installment is a new standalone purchase.

---

## 9. Refund reconciliation

Try to link refund to original purchase.

Signals:

- amount;
- merchant;
- card;
- date distance;
- provider reference;
- installment number.

If exact link is not known, keep refund as separate credit with unresolved original linkage.

Never convert uncertainty into an invented purchase relation.

---

## 10. Duplicate handling

### Proven duplicate
Same source external ID/hash/statement row identity.

Auto-deduplicate.

### Probable duplicate
Same merchant/date/amount but no definitive identity.

Keep both until reviewed.

User choice:

- same transaction;
- both real.

Do not create a global rule from a single duplicate decision.

---

## 11. Reconciliation preview

Before commit, show:

- rows parsed;
- confirmed matches;
- probable matches;
- new events;
- statement-only financial costs;
- unmatched provisional events;
- duplicates;
- arithmetic status.

Example:

> 154 rows  
> 147 confirmed matches  
> 3 new events  
> 2 probable duplicates  
> 1 interest charge  
> 1 refund  
> Statement arithmetic: PASS

---

## 12. Commit semantics

Prefer a transactional database commit.

At commit:

1. create/update source events;
2. link matches;
3. create new transactions;
4. promote provisional to posted;
5. create financial cost transactions;
6. update installment plan;
7. create pending reviews;
8. write change/audit record;
9. mark statement committed.

If any invariant fails, rollback.

---

## 13. Reconciliation metrics

Track:

- exact match rate;
- probable match rate;
- new statement-only rate;
- false duplicate correction rate;
- unmatched provisional count;
- reconciliation time;
- parser mismatch frequency.

---

## 14. Acceptance criteria

- same purchase from e-mail + statement appears once in economic spending;
- repeat statement import changes nothing;
- user classifications survive match;
- statement arithmetic must pass or explicitly remain unresolved;
- probable duplicate is never silently deleted;
- refund/card-payment/installment semantics remain intact after reconciliation.
