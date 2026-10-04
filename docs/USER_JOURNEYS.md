# User Journeys

These journeys define the intended product behavior before implementation.

---

## Journey 1 — First-time setup

### Goal
Get useful results without building a budget taxonomy manually.

### Flow
1. User signs in.
2. Creates/accepts one household.
3. Adds household people: e.g. Selçuk, Büşra.
4. Optionally names existing cards/accounts as they are discovered.
5. Uploads one recent statement.
6. System detects bank, period and cards.
7. System parses/imports as a draft.
8. System asks only unresolved questions.
9. User reaches dashboard.

### Must not happen
- category tree wizard;
- 30-minute account setup;
- forcing all merchants to be classified before showing value.

---

## Journey 2 — Routine card purchase arrives by e-mail

### Example
A known grocery merchant sends a bank notification.

### Flow
1. E-mail ingestion receives message.
2. Provider message ID is checked for idempotency.
3. Bank parser extracts masked card, amount, merchant and event time.
4. Merchant alias normalizes the name.
5. Existing user rule applies:
   - category: Groceries;
   - necessity: Required;
   - owner: Household.
6. Provisional transaction is created.
7. No user notification is required.

### Success
The user does nothing.

---

## Journey 3 — Unknown purchase arrives by e-mail

### Flow
1. Transaction is extracted.
2. No strong rule/merchant history exists.
3. AI may suggest category with confidence.
4. If confidence is below threshold or necessity/owner remain uncertain, item enters Pending.
5. Home shows “1 item needs your input.”
6. User chooses the needed value.
7. App optionally asks:
   > Apply this to future transactions from this merchant?
8. Rule is saved if approved.

### Success
One decision improves future automation.

---

## Journey 4 — User sends a PDF statement to ChatGPT

### Goal
The user does not need to open the finance app.

### Flow
1. User attaches statement in ChatGPT.
2. User says:
   > “Add this statement to my finance app.”
3. ChatGPT invokes statement import tool with supported file reference.
4. Finance backend creates an import job and parses the file.
5. ChatGPT retrieves preview:
   - bank;
   - period;
   - row count;
   - matches;
   - new items;
   - conflicts;
   - fees/refunds/installments.
6. If safe to commit, ChatGPT asks/acts according to permission policy.
7. Import is committed.
8. ChatGPT reports:
   > “Statement reconciled. 147 matched, 3 new, 2 items need review.”
9. Pending items remain visible in both app and ChatGPT.

### Failure mode
If file transfer is not supported on the active ChatGPT surface, the integration adapter must fall back to a supported upload endpoint without changing the core Finance API.

---

## Journey 5 — Monthly statement reconciliation after e-mail ingestion

### Flow
1. Most purchases already exist provisionally.
2. User uploads monthly statement.
3. Reconciliation matches posted statement rows to provisional events.
4. Matching transactions receive authoritative statement metadata.
5. User category/owner/necessity edits are preserved.
6. Statement-only items are added.
7. E-mail-only items remain unresolved until explained.
8. Duplicate candidates enter review.
9. Accounting totals are validated.
10. Statement closes successfully only if invariants pass or explicit reviewed adjustments explain the difference.

### Success
No double-counting.

---

## Journey 6 — Cash expense by conversation

User says:

> “Büşra paid 850 TL cash at the greengrocer today, household spending.”

Flow:

1. ChatGPT parses intent.
2. Searches/normalizes merchant if relevant.
3. Calls add_transaction with:
   - amount 850;
   - currency TRY;
   - cash account;
   - payment actor Büşra;
   - owner Household;
   - category Groceries/Greengrocer;
   - proposed necessity Required.
4. Backend applies validation.
5. Audit entry records ChatGPT actor.
6. ChatGPT confirms.

If one field is uncertain, ChatGPT asks only for that field.

---

## Journey 7 — Correct an existing transaction

User says:

> “The 4,393 TL Amazon transaction was the shelf for the children’s room. Make it household, home goods, necessary/flexible.”

Flow:

1. ChatGPT searches exact candidate(s).
2. If one strong match exists, retrieves detail.
3. Calls update/classify operation.
4. Audit records old/new state.
5. Optionally asks if future Amazon transactions should follow the same rule.
6. By default, do **not** create an Amazon-wide rule unless user explicitly agrees; Amazon is too heterogeneous.

---

## Journey 8 — Merchant-wide rule

User says:

> “All Toyzz Shop transactions should be children / discretionary.”

Flow:

1. Search matching merchant aliases.
2. Preview affected transactions and amount.
3. Apply merchant rule.
4. Reclassify eligible historical transactions if user asked for “all”.
5. Preserve any explicit transaction overrides that have higher priority.
6. Record one change set in audit log.

---

## Journey 9 — Installment purchase

A 30,000 TL purchase is divided into 3 installments.

System records:

- original purchase economic amount: 30,000;
- installment plan: 3 × 10,000;
- each statement charge;
- remaining obligation.

User can view:

### Economic spending
30,000 in purchase month.

### Current card burden
10,000 on each statement month.

The dashboard must make clear which view is active.

---

## Journey 10 — Refund

A refund arrives for a prior purchase.

Flow:

1. System detects refund event.
2. Attempts to link it to original transaction using reference/merchant/amount/date.
3. If strong match:
   - links refund;
   - updates net economic cost.
4. If ambiguous:
   - pending item asks user which purchase was refunded.
5. Refund is never counted as a new positive expense.

---

## Journey 11 — Card payment

A 50,000 TL bank-to-card payment arrives.

Flow:

1. Event classified as CARD_PAYMENT.
2. It reduces card liability/cash account balance as appropriate.
3. It does not appear in economic spending charts.
4. It may appear in cash-flow/debt views.

---

## Journey 12 — Duplicate suspicion

Two transactions share similar merchant/date/amount.

Flow:

1. Match engine assigns probability.
2. If source IDs prove same event: auto-merge.
3. If merely similar: keep both and surface a duplicate review.
4. User chooses:
   - same transaction;
   - both real.
5. Decision becomes evidence for reconciliation, not a generic merchant rule.

---

## Journey 13 — “Why did spending increase?”

User asks:

> “Why did September increase?”

Flow:

1. System uses economic-spending view.
2. Compares with selected baseline.
3. Decomposes delta by:
   - category;
   - merchant;
   - one-off events;
   - travel/event tags;
   - household/person;
   - recurring vs non-recurring.
4. Returns top contributors rather than generic advice.

Example response structure:

> September was 24,600 TL higher.  
> 10,400 TL came from a motorcycle installment obligation, 6,400 TL from personal care, and 5,700 TL from travel-related dining/transport. Normal household groceries were roughly flat.

---

## Journey 14 — Undo ChatGPT change

User says:

> “Undo what you just changed.”

Flow:

1. ChatGPT retrieves last eligible change set it initiated in current context/user account.
2. Backend validates that later conflicting edits do not make reversal unsafe.
3. Reversal is applied.
4. New audit entry links to reverted change.

If unsafe:

> “That transaction was edited again afterward. I can show the differences before reverting.”

---

## Journey 15 — Parser mismatch

Statement closing balance does not reconcile.

Flow:

1. Import remains draft.
2. System shows:
   - expected closing balance;
   - parsed closing balance;
   - difference;
   - likely suspicious rows.
3. No silent commit.
4. User can review or provide a corrected statement.
5. Parser bug becomes a regression fixture if appropriate.

---

## Journey 16 — New model / new ChatGPT conversation

### Goal
No loss of financial behavior.

Flow:
1. New conversation starts with no chat memory assumptions.
2. ChatGPT queries finance plugin tools.
3. Rules, household structure, merchants and prior transactions remain available from application state.
4. User receives consistent classifications.

### Success
Application data, not chat memory, is the continuity layer.

---

## Journey 17 — Export and exit

User requests data export.

Flow:

1. Export transaction ledger and supporting metadata in portable format.
2. Include rules, categories and audit identifiers where practical.
3. Exclude secrets/tokens.
4. User can delete account/data according to retention policy.

The product must not trap the user through proprietary storage.
