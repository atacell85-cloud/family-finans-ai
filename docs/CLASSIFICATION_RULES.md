# Classification Rules

## 1. Purpose

Classification should become more accurate and less intrusive over time.

The system must distinguish between:

- deterministic user intent;
- learned/confirmed patterns;
- provider metadata;
- AI suggestion.

---

## 2. Classification dimensions

A transaction can have:

### Category
Examples:
- Groceries;
- Dining;
- Health;
- Transportation;
- Children;
- Home;
- Personal care;
- Travel;
- Digital/subscriptions;
- Taxes/fees.

### Subcategory
Optional and progressively refined.

### Necessity
- required;
- necessary_flexible;
- discretionary;
- unknown.

### Reducible now
- yes;
- no;
- unknown.

### Economic owner
- household;
- person;
- future dependent/entity.

### Tags
Examples:
- Çanakkale trip;
- school;
- medical;
- one-off;
- recurring.

These dimensions must remain independent.

---

## 3. Precedence

Highest to lowest:

1. transaction-specific user override;
2. explicit user-created rule;
3. confirmed merchant mapping/history;
4. deterministic system mapping;
5. provider/bank category metadata;
6. AI inference;
7. unknown.

Lower-priority evidence must never silently overwrite higher-priority evidence.

---

## 4. Rule types

### Merchant exact rule
Example:
> canonical merchant = Özkuruşlar  
> category = Groceries  
> necessity = Required  
> owner = Household

### Merchant alias rule
Maps raw processor names to canonical merchant.

### Merchant + context rule
Example:
> merchant = Shell  
> account = family car card  
> category = Fuel

### Date/event scoped rule
Example:
> dates 18–21 Sep + location/tag = Çanakkale trip  
> category family = Travel

### Amount/context rule
Use cautiously.

### Source-specific rule
Example:
> a certain bank description pattern means card payment.

System financial-type rules are separate from user spending-preference rules.

---

## 5. What should not become a broad rule automatically

Heterogeneous merchants:

- Amazon;
- Trendyol;
- Hepsiburada;
- department stores;
- payment processors.

A single user correction should not create a merchant-wide category rule unless explicitly approved.

---

## 6. Rule creation UX

After manual correction:

> Apply this to future transactions from Özkuruşlar?

If yes:
- create rule;
- record author=user;
- record source correction;
- optionally offer historical reclassification.

Do not expose a technical DSL to normal users.

---

## 7. Historical application

When user says:

> “All Toyzz Shop transactions are children / discretionary.”

System should:

1. resolve aliases;
2. find eligible historical transactions;
3. exclude transaction-specific overrides unless user explicitly requests override;
4. preview affected count/amount;
5. apply as one change set;
6. create future rule.

---

## 8. AI fallback

AI is called only when deterministic layers cannot resolve needed fields.

Provide minimal context:

- raw/canonical merchant;
- amount;
- currency;
- date;
- bank-provided type/category;
- limited nearby/context history;
- household rule vocabulary.

Expected response:
- proposed category;
- proposed necessity;
- optional owner suggestion;
- confidence per field;
- short machine-readable rationale.

If confidence below threshold:
- leave unknown;
- create pending review.

---

## 9. Confidence policy

Thresholds should be empirical.

Example starting policy:

- ≥ 0.95: auto-apply AI suggestion only for low-risk classification fields;
- 0.75–0.95: suggest in pending review;
- < 0.75: do not preselect strongly.

But user-authored rules may auto-apply regardless of AI.

Never use confidence to bypass a conflicting user rule.

---

## 10. Explainability

Transaction detail should show one simple provenance string:

Examples:

- “User rule: Özkuruşlar”
- “Confirmed from 12 prior transactions”
- “Bank metadata”
- “AI suggestion · 82%”
- “Unknown”

---

## 11. Rule conflicts

If multiple rules of equal priority disagree:

- do not guess;
- mark conflict;
- pending review;
- show both rules and scopes.

Resolve by:
- specificity;
- explicit user update;
- disabling/deleting obsolete rule.

---

## 12. Versioning

Rules should include:

- created_at;
- updated_at;
- author;
- active flag;
- scope;
- priority/specificity metadata;
- audit history.

AI model upgrades should not silently rewrite rules.

---

## 13. Testing

Must cover:

- precedence;
- alias resolution;
- explicit override protection;
- heterogeneous merchant no-auto-rule behavior;
- context scoping;
- historical reclassification;
- conflict state;
- AI low-confidence fallback;
- user-rule persistence across import/reconciliation.

---

## 14. Acceptance criteria

- a corrected routine merchant stops asking repeatedly;
- AI cannot override a user-confirmed transaction;
- merchant-wide rule creation requires explicit intent;
- broad ambiguous merchants remain context-sensitive;
- rule changes are auditable;
- historical bulk change can be previewed and undone.
