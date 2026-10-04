# Open Questions

These questions must be resolved deliberately. They are not blockers to documentation, but some block implementation phases.

---

## Product / UX

### Q1 — Exact primary dashboard metric
Should the headline be:
- “real economic spending”; or
- switchable economic/cash view with economic as default?

**Proposal:** economic spending default; debt/cash snapshot secondary.

### Q2 — Terminology
Is “Necessary / Flexible” the best Turkish/user-facing label?

Need final language testing.

### Q3 — Personal vs household ownership
Should dependent/child ownership exist in MVP or use tags/categories initially?

**Proposal:** household/person only in schema, extensible to dependents later.

### Q4 — Pending thresholds
How much uncertainty is acceptable before asking the user?

Must be tuned with real corrections.

### Q5 — Notification strategy
Should the app proactively notify for pending items or only show count?

**Proposal:** avoid noisy per-transaction notifications; batch or monthly unless material anomaly.

---

## Credit cards

### Q6 — Economic recognition policy for installments
Default full purchase at original purchase date seems analytically best, but historical statement-only imports may not reveal original total reliably.

Need rule:
- if original purchase known → full economic recognition;
- if only occurrence known → do not invent original total; represent known installment and plan estimate separately.

### Q7 — How to handle pre-statement card activity?
Provisional e-mail event may be cancelled/reversed.

Need status and aging policy.

### Q8 — Reward accounting
Should rewards reduce category spend or remain separate offset?

**Proposal:** preserve gross and net; default spending chart gross economic cost with explicit reward offset option.

---

## Ingestion

### Q9 — First bank/source
Which bank and exact statement/e-mail format is the first supported production adapter?

### Q10 — Gmail integration mechanism
Polling, Apps Script relay, Gmail API watch/push, or another low-maintenance approach?

Decision should optimize privacy + operational simplicity.

### Q11 — Raw e-mail retention
Default zero/short-term retention?

### Q12 — Statement file retention
Delete after successful reconciliation or keep encrypted/private for audit?

User preference may control.

---

## ChatGPT / plugin

### Q13 — Current file handoff capability
Can the active ChatGPT plugin/MCP surface pass the attached PDF directly to our tool today?

Must be tested, not assumed.

### Q14 — Current write permission UX
Which tool actions require platform confirmation and which can run under the user’s chosen plugin permission settings?

### Q15 — Plugin hosting
Where should MCP server run for simplest reliable authentication and file handling?

### Q16 — ChatGPT-only vs embedded chat
Do we need chat inside the PWA at all?

**Proposal:** no for MVP. Let ChatGPT be the conversational surface; keep app focused.

---

## Security

### Q17 — Repository visibility
Repository is currently public unless changed by owner. Should it become private before implementation?

**Strong proposal:** private, because accidental fixture/log/config commits become more consequential in a finance project.

### Q18 — Statement storage retention
Need final policy before production.

### Q19 — Export/delete semantics
How long are backups retained after account deletion?

---

## Technical

### Q20 — Monorepo tooling
pnpm workspaces + Turborepo, or plain pnpm workspaces?

**Proposal:** plain pnpm workspaces first; add task runner only if needed.

### Q21 — Background job mechanism
Supabase Edge Functions/cron, managed queue, or small worker service?

Start simple; choose after Gmail/parser workload known.

### Q22 — Offline support
Is offline write/sync actually required, or just installable PWA + cached read shell?

**Proposal:** no complex offline ledger sync in MVP.

### Q23 — Analytics materialization
Postgres views vs materialized views?

**Proposal:** ordinary queries/views first.

### Q24 — Money representation
Integer minor units vs numeric/decimal.

Need one consistent convention before schema.

### Q25 — Multi-currency conversion source
When/if base currency analytics need FX rates, which rate/source/date is authoritative?

Defer until foreign-currency use case appears.

---

## Process

### Q26 — Second-model review
Run the master plan through Claude using `CLAUDE_REVIEW_PROMPT.md`, save raw response, and triage.

### Q27 — UI prototype fidelity
Use low-fidelity wireframes first or immediately build clickable prototype?

**Proposal:** low-fi flow + one clickable mobile prototype before backend expansion.
