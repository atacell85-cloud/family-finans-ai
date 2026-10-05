# Decisions

Architecture/Product Decision Log.

Use IDs `ADR-XXX`. Do not rewrite history; append superseding decisions.

---

## ADR-001 — AI-first, not AI-dependent

**Status:** Accepted

Core ledger, statement semantics, rules and reconciliation must work without an LLM. AI provides conversational control, fallback classification and explanation.

**Reason:** Financial correctness cannot depend on model availability or changing provider behavior.

---

## ADR-002 — UI and AI share domain operations

**Status:** Accepted

The PWA and ChatGPT/MCP adapter must call the same business/domain layer.

**Reason:** Prevents behavior drift and duplicate rule implementations.

---

## ADR-003 — No raw SQL tool for AI

**Status:** Accepted

ChatGPT receives bounded domain tools only.

**Reason:** Limits blast radius, improves auditability and allows validation/authorization.

---

## ADR-004 — Statement as reconciliation authority

**Status:** Accepted

Bank e-mail notifications are provisional. Posted statement rows are stronger evidence for card activity.

**Reason:** Notification e-mails can be missing, delayed, reversed or formatted inconsistently.

---

## ADR-005 — Economic spending separated from debt/cash movement

**Status:** Accepted

Dashboards must provide distinct semantic views.

**Reason:** Statement balance and monthly spending are not equivalent.

---

## ADR-006 — User rules stored in application DB

**Status:** Accepted

Persistent classification behavior is not stored only in chat/model memory.

**Reason:** Rules must survive model/provider/conversation changes.

---

## ADR-007 — Four necessity states

**Status:** Accepted

UI classification:
- Required;
- Necessary/Flexible;
- Discretionary;
- Unknown.

“Reducible now?” remains a separate dimension.

---

## ADR-008 — Payment actor separate from economic owner

**Status:** Accepted

A cardholder/person and the beneficiary/owner of spending are different concepts.

---

## ADR-009 — Adapter-based bank parsing

**Status:** Accepted

Use bank-specific deterministic adapters with generic/AI fallback rather than one universal AI parser.

**Reason:** Improves testability, reliability and cost.

---

## ADR-010 — Pending inbox is primary review surface

**Status:** Accepted

The user reviews uncertainty and conflicts, not every transaction.

---

## ADR-011 — Supabase/Postgres baseline

**Status:** Proposed / to validate during implementation spike

Use Supabase for Postgres/Auth/Storage/RLS unless a concrete limitation emerges.

---

## ADR-012 — Next.js/TypeScript PWA baseline

**Status:** Proposed / to validate

Chosen for shared TypeScript domain/MCP ecosystem and low operational overhead.

---

## ADR-013 — ChatGPT capability spike before full MCP implementation

**Status:** Accepted

Verify actual read/write/file behavior in the user’s current ChatGPT environment before building the entire integration.

**Reason:** Platform capabilities can evolve and may vary by plan/surface.

---

## ADR-014 — No real financial fixtures in Git

**Status:** Accepted

Use synthetic or irreversibly anonymized fixtures only.

---

## ADR-015 — Public SaaS features excluded

**Status:** Accepted

Initial product is a personal household system. Multi-tenant commercial onboarding is not an MVP concern.


---

## ADR-016 — V2: Order & Item Intelligence

**Status:** Accepted for V2 / Explicitly deferred from MVP

The application will eventually expand from household financial intelligence into item-level consumption intelligence by linking financial transactions to e-commerce/order evidence such as order-confirmation e-mails.

### V2 vision

A financial transaction such as:

> 05.12.2026 — Migros — 1.200 TL

may be linked to an order e-mail containing item-level detail such as:

> Dana kıyma — 1 kg — 620 TL  
> Pınar süt — 2 adet — 110 TL  
> Other items / delivery / discount lines

The financial transaction remains the authoritative payment event. Order and item data enrich it rather than replacing it.

The intended conceptual relationship is:

```text
Transaction
  ↕
Order
  ↕
Shipment / Invoice
  ↕
Order Line
  ↕
Canonical Product
```

The relationship must not assume that one card transaction always equals one order. V2 must allow many-to-many reconciliation because real commerce can contain:

- one order charged in multiple transactions;
- one transaction covering multiple seller/order components;
- partial shipment;
- weighted-product price differences;
- coupons and promotional discounts;
- partial refunds;
- cancellations/reversals;
- marketplace sellers;
- delivery/service fees.

### Item-level capabilities planned for V2

- ingest order-confirmation e-mails from selected commerce providers;
- extract order number, merchant, seller, product, brand, quantity, unit, unit price, line total, discount, delivery and refund information;
- reconcile orders with existing financial transactions;
- attach an order summary to the transaction-detail view;
- drill down from financial category → merchant → order → item;
- normalize equivalent product names into canonical products;
- maintain historical product/unit-price series;
- support item-, brand-, merchant- and category-level analytics;
- distinguish price inflation from quantity/consumption change where the available data permits it;
- detect recurring purchase patterns and product replenishment cadence;
- support future shopping-agent workflows such as comparison, basket optimization and—only with explicit user authorization—purchase preparation/execution.

Example analytical questions V2 should eventually answer:

- How much was spent on red meat in the last 12 months?
- What was the unit-price history of the same milk product?
- Is market spending rising because prices increased or because more products were purchased?
- Which merchant is usually cheaper for a recurring basket?
- When was a specific product last bought and at what price?
- Which routine household products are likely to need replenishment soon?

### Commercial/data-use boundary

Item-level purchase history is highly sensitive behavioral data.

The product must **not** assume that raw or identifiable user shopping data can be sold or commercially reused.

Any future external/commercial use must be a separate product/policy decision and must require, as applicable:

- explicit opt-in consent;
- clear purpose limitation;
- privacy/legal review;
- provider/API/mail-platform terms review;
- aggregation/anonymization where appropriate;
- revocation/deletion mechanisms;
- transparent disclosure to the user.

A preferable long-term commercial direction is value created **for the user**—for example basket optimization, price comparison or agent-assisted shopping—rather than monetizing identifiable purchase histories.

### V1 architectural implications

V2 is **not** part of the MVP and no V2 commerce integrations should be implemented during V1.

However, V1 architecture must avoid choices that would make V2 expensive or require rewriting the ledger.

Therefore V1 should preserve these boundaries:

1. **Transaction remains separate from evidence/source records.**  
   Future order e-mails must be able to become additional source/evidence entities without altering the core transaction model.

2. **Merchant is an independent dimension, not a category node.**  
   This allows analysis such as Market → Migros and also merchant-specific histories without corrupting category taxonomy.

3. **Classification dimensions remain independent.**  
   Category, owner/person, necessity, recurrence, payment actor, payment method and future order/product metadata must not be encoded into a single category tree.

4. **Transaction relationships must be extensible.**  
   The data model should be able to link one transaction to zero, one or multiple external/order entities later.

5. **Source types must be extensible.**  
   V1 must not hard-code source types in a way that prevents future sources such as `commerce_email`, `order_api`, `invoice` or `receipt`.

6. **Raw-source parsing and financial truth remain separate.**  
   AI may extract order/item data, but extracted e-mail text does not become authoritative financial truth without reconciliation to the ledger.

7. **Stable identifiers and provenance are mandatory.**  
   Future order/product records will need to trace extracted facts back to the originating e-mail/API/document.

8. **Analytics should be dimension-driven.**  
   V1 analytics should avoid assumptions that aggregation ends at transaction category; V2 will add merchant/order/item drill-down beneath the same analytical framework.

### Explicit V1 non-goals

Do not build during V1:

- Amazon/Migros/Trendyol order parsers;
- product/SKU catalog;
- product normalization;
- unit-price history;
- consumption forecasting;
- price-comparison engine;
- shopping-agent execution;
- commercial sale/export of behavioral purchase data.

These belong to V2 after the core financial ledger, reconciliation, UX and ChatGPT workflows are proven.

**Reason:** The V2 direction materially affects how V1 should preserve transaction provenance, merchant identity, independent classification dimensions and extensible relationships. Recording the direction now prevents avoidable architectural lock-in without expanding the V1 implementation scope.
