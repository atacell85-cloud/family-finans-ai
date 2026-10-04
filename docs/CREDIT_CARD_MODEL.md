# Credit Card Model

## 1. Problem

Credit-card statements mix spending with liability management.

A correct system must distinguish:

- the purchase;
- installment schedule;
- statement charge;
- payment;
- carried balance;
- interest/fees/taxes;
- refund;
- reward/discount.

A statement balance is not the same as monthly spending.

---

## 2. Core concepts

### Purchase
Economic acquisition of a good/service.

### Installment plan
Schedule created from a purchase.

### Installment occurrence
One scheduled amount appearing on a statement.

### Statement
Period summary from issuer.

### Card payment
Settlement of card liability.

### Carryover
Unpaid prior statement balance.

### Financing cost
Interest/fees/taxes created by carrying/using credit.

---

## 3. Dual reporting views

### 3.1 Economic spending view

Recognize the purchase according to chosen economic policy.

Recommended default:

For normal consumer installment purchases, recognize full original purchase amount on purchase date for “what did we buy?” analytics.

### 3.2 Liability/cash view

Recognize each installment as it becomes due on statements for “what must we pay?” analytics.

Both views are shown distinctly.

---

## 4. Example

Purchase:
- 30,000 TL;
- 3 installments;
- purchase date: July 10.

Economic view:
- July spending +30,000 TL.

Debt view:
- July/Aug/Sep statement obligation +10,000 TL each, subject to issuer timing.

The user may filter economic charts to “cash burden” if desired, but the system must not mix the two silently.

---

## 5. Suggested installment plan fields

- id;
- household_id;
- payment_instrument_id;
- merchant_id;
- original_transaction_id;
- original_purchase_date;
- original_amount;
- currency;
- total_installments;
- installment_amount_expected;
- installments_posted;
- remaining_installments;
- remaining_amount_estimated;
- status;
- source/provenance.

Occurrence fields:
- statement_id;
- sequence_number;
- amount;
- posted_date;
- transaction_id.

---

## 6. Partial/irregular installments

Do not assume every plan is equal installments.

Support:
- final rounding difference;
- bank fees;
- refunded installment;
- accelerated/early closure;
- merchant cancellation.

Remaining amount should be derived from known occurrences and issuer hints, not blindly total_count × first_amount.

---

## 7. Statement summary fields

Where available:

- previous balance;
- period charges;
- payments;
- interest;
- fees/taxes;
- closing balance;
- minimum payment;
- due date;
- next statement date.

These are statement facts, not expense categories.

---

## 8. Card hierarchy

Support:

- account;
- primary card;
- supplementary card(s);
- cardholder/person.

A supplementary card transaction can share statement account but have distinct payment actor.

---

## 9. Refunds

Refund may:
- reverse full original purchase;
- reverse one installment;
- create statement credit;
- arrive in a later period.

Link when evidence permits.

Economic view should reflect net effect without pretending the refund is income.

---

## 10. Rewards and discounts

Examples:
- points;
- statement credit;
- campaign discount.

Keep a separate reward/discount type.

Reporting may show:
- gross purchase;
- reward offset;
- net cash cost.

Do not hide gross consumption by subtracting rewards everywhere.

---

## 11. Interest and tax

Interest/fees/taxes are real financial costs.

They should:
- appear in financing-cost analysis;
- not be attributed to merchant purchase category;
- be clearly separated from consumption spending.

---

## 12. Balance carryover

Carryover is a liability state, not a current-month purchase.

It may influence:
- debt dashboard;
- financing cost;
- payment burden.

It must not inflate current economic spending.

---

## 13. Payment

Card payment reduces liability.

If money comes from owned bank account:
- bank account cash decreases;
- card liability decreases;
- household net spending does not increase.

This prevents double counting.

---

## 14. Reconciliation checks

A statement adapter should verify, according to bank sign rules:

```text
previous balance
+ purchases/charges
+ interest/fees/taxes
- refunds/credits
- payments
= closing balance
```

Use exact issuer semantics when field labels differ.

---

## 15. Dashboard questions this model must answer

- What did we buy this month?
- How much card debt is due?
- How much of this statement comes from prior-period debt?
- How much interest/fee did we pay?
- How much future installment burden remains?
- Which person’s card generated the charge?
- Which spending belongs to household vs personal?
- What would next month’s obligation be even if we bought nothing new?

---

## 16. Acceptance criteria

- card payment never appears as purchase spending;
- carryover never appears as new spending;
- installment purchase and installment obligation can be viewed separately;
- remaining installment burden is consistent with posted occurrences;
- refunds reduce the correct economic cost where linkage is known;
- financing costs remain visible but separate from consumption.
