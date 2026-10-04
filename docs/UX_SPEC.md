# UX Specification

## 1. UX objective

The user should feel that the system is maintaining the ledger on their behalf.

The UI exists for four jobs:

1. understand;
2. review uncertainty;
3. correct exceptions;
4. add missing off-system activity.

It should not feel like bookkeeping software.

---

## 2. Primary navigation

Keep primary navigation to four destinations:

### Home
“What happened this month?”

### Activity
Search, filter, inspect and edit transactions.

### Add
Quick add, statement import, file import.

### Insights
Trends, category/person analysis, debt/installments, comparisons.

Settings, accounts, rules, import sources and security live in secondary navigation.

ChatGPT is not required to be a separate tab. A compact “Ask or change something…” control may be available globally.

---

## 3. Home screen hierarchy

The first viewport should answer, in order:

1. **Real spending this month**
2. **Change vs previous comparable period**
3. **Required / flexible / discretionary / unknown**
4. **Pending items**
5. **Household vs personal**
6. **Key drivers**
7. **Debt/installment snapshot**

Suggested content:

> October  
> Real spending: 142,350 TL  
> +12% vs September  
>
> Required 71,200  
> Necessary/Flexible 32,800  
> Discretionary 31,900  
> Unknown 6,450  
>
> 4 items need your input

Do not put account balances, transaction tables and ten charts above the fold.

---

## 4. Pending inbox

### 4.1 Purpose

A single list for all exceptions:

- unknown merchant/category;
- uncertain necessity;
- unclear owner;
- probable duplicate;
- reconciliation mismatch;
- parser uncertainty;
- rule conflict.

### 4.2 Card design

Example:

> AMAZON — 4,393 TL  
> 7 Sep · Card •••• 2825  
> I could not determine what this was.

Primary quick actions should match the decision needed.

For necessity:

- Required
- Flexible
- Discretionary
- Details

For duplicate:

- Same transaction
- Both are real
- Details

For owner:

- Household
- Selçuk
- Büşra
- Details

### 4.3 One decision at a time

Do not force the user to fill category, owner, necessity and notes simultaneously unless all are required.

Resolve the uncertainty that caused the item to enter the inbox.

### 4.4 Rule creation

After correction:

> Use this classification for future Özkuruşlar transactions?

Options:

- Yes
- Not now

No separate rule editor is required for ordinary use.

---

## 5. Quick add

### 5.1 Natural language first

Input:

> “850 TL manav, Büşra nakit ödedi”

Preview:

> 850 TL  
> Groceries · Required  
> Paid by Büşra · Owner: Household  
> Cash · Today

Actions:

- Save
- Edit details

### 5.2 Structured fallback

If parsing is uncertain, ask only for missing fields.

Example:

> “850 market”

System asks:

> Who paid?

rather than opening a full form.

### 5.3 Repeat shortcuts

Frequently used quick actions may appear:

- Cash groceries
- Cleaner
- Allowance
- Fuel

Only after actual usage data supports them.

---

## 6. Statement import UX

### Step 1 — Select file

User chooses PDF/CSV/XLSX.

### Step 2 — Detect

Show:

> VakıfBank Credit Card Statement  
> Period: Sep 2026  
> 4 cards found  
> 154 statement lines

### Step 3 — Reconcile

System runs parsing, typing, matching and rules.

### Step 4 — Summary

Show only outcome:

> 147 matched existing activity  
> 3 new posted transactions  
> 2 likely duplicates  
> 1 interest charge  
> 1 refund  
> 6 items need review

Primary action:

**Review 6 items**

Secondary:

**View import details**

### Step 5 — Commit

If no material conflict:

> Statement reconciled successfully.

If accounting invariant fails:

> This statement does not reconcile. No silent import.

Provide a clear diagnostic and preserve the import job for review.

---

## 7. Transaction detail

Default fields:

- merchant;
- amount;
- date;
- category;
- necessity;
- owner;
- payment method;
- type;
- installment status if relevant;
- source badge.

Expandable advanced section:

- source event(s);
- parser;
- raw description;
- statement period;
- posting date;
- reconciliation status;
- confidence;
- rule applied;
- audit history.

---

## 8. Bulk changes

Bulk changes should usually originate from search or natural language.

Example:

> “Mark restaurants from the Çanakkale trip as Travel / Discretionary.”

Before committing a broad mutation, preview:

> 12 transactions · 8,430 TL  
> Change category → Travel / Dining  
> Necessity → Discretionary

Then apply as one auditable change set.

---

## 9. Debt/installment UX

This view must distinguish:

### Current statement
- statement balance;
- previous balance;
- payments;
- interest/fees;
- minimum payment;
- due date.

### Future obligations
- next month known installments;
- following 2–3 months;
- total remaining installment obligation.

### Economic context
A badge can indicate:

> Purchase occurred in July; this is installment 3/6.

This prevents the user from interpreting the current installment as a new purchase.

---

## 10. Charts

Charts are for decisions, not decoration.

MVP charts:

1. monthly real-spending trend;
2. necessity distribution;
3. top category trend;
4. household vs personal;
5. person-level personal spending;
6. remaining installment schedule.

Avoid pie-chart overload. Tables or ranked bars are preferable when exact comparison matters.

---

## 11. Search

Support structured filters and natural language.

Examples:

- Amazon in September;
- Büşra personal over 2,000 TL;
- all travel dining;
- unknown transactions;
- all installments remaining after December;
- card ending 5611;
- discretionary child spending.

Natural language search should translate into the same domain query API used by the UI.

---

## 12. Language and tone

Use neutral analytical wording.

Preferred:

- “Needs review”
- “Unknown”
- “Discretionary”
- “This month increased mainly because…”
- “This transaction was matched to…”

Avoid:

- “Bad spending”
- “You overspent”
- “Waste”
- “Guilty purchase”

unless the user defines that terminology.

---

## 13. Feedback states

### Good automation
Keep it quiet.

Do not celebrate every classified transaction.

### Uncertainty
Be explicit:

> “I’m not confident enough to classify this.”

### Failure
Explain next action:

> “The statement total does not match the parsed rows. I kept the import as a draft.”

### AI write
Provide subtle confirmation:

> “Updated 3 Toyzz Shop transactions and saved a future rule.”

---

## 14. Accessibility and interaction

- minimum touch targets appropriate for mobile;
- no color-only meaning;
- text labels for classification states;
- keyboard-safe quick-add field;
- responsive tables;
- large amount typography with clear sign;
- locale-aware number/date formatting;
- dark mode optional, not a blocker.

---

## 15. UX acceptance metrics

A release should be rejected if:

- routine statement import requires reviewing every row;
- adding a cash expense takes more than a few seconds;
- a user cannot see why a transaction is pending;
- a user cannot reverse an AI change;
- the home screen requires scrolling before showing real spending and pending count;
- a previously confirmed common merchant repeatedly asks the same question.

Target:

- < 5 manual review decisions per 100 routine transactions at maturity;
- < 10 minutes monthly cleanup;
- common correction ≤ 2 taps or one natural-language instruction.
