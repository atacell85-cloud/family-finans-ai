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
