# Claude Review Prompt

Copy the prompt below into Claude together with the repository planning documents, especially:

- README.md
- docs/MASTER_PLAN.md
- docs/PRODUCT_PRINCIPLES.md
- docs/UX_SPEC.md
- docs/USER_JOURNEYS.md
- docs/ARCHITECTURE.md
- docs/DATA_INGESTION.md
- docs/STATEMENT_RECONCILIATION.md
- docs/CLASSIFICATION_RULES.md
- docs/CREDIT_CARD_MODEL.md
- docs/CHATGPT_PLUGIN_MCP.md
- docs/SECURITY_PRIVACY.md
- docs/TEST_STRATEGY.md
- docs/ROADMAP.md
- docs/ACCEPTANCE_CRITERIA.md
- docs/DECISIONS.md
- docs/OPEN_QUESTIONS.md

Do **not** ask Claude to rewrite the project yet. Ask for an adversarial review first.

---

## Prompt

You are reviewing the architecture and product plan for a private, AI-first household finance application before implementation.

Act simultaneously as:

1. a principal fintech software architect;
2. a senior mobile-product/UX reviewer;
3. a payment/credit-card domain analyst;
4. a security/privacy engineer;
5. an AI-agent/MCP tool-safety reviewer;
6. a pragmatic engineering manager whose job is to prevent overengineering and wasted time.

The user’s priority is extremely important:

> This must be highly user-friendly and low-maintenance. The user does not want to spend significant time operating or developing the system. The system should ingest financial activity automatically where possible and ask only about uncertainty.

The product concept is:

- mobile-first PWA;
- household finance rather than business accounting;
- bank notification e-mail ingestion;
- PDF/CSV/XLSX statement import;
- statement reconciliation;
- manual/cash entry;
- household/person ownership;
- required / necessary-flexible / discretionary / unknown classification;
- separate “reducible now?” concept;
- installment/debt modeling;
- merchant rules;
- pending-review inbox;
- audit/undo;
- charts/analysis;
- ChatGPT/plugin/MCP as a first-class read/write/import interface;
- persistent rules stored in the app, independent of model memory;
- Supabase/Postgres baseline;
- strong privacy requirements.

A crucial financial requirement is that a credit-card statement balance is NOT treated as monthly spending. The system must distinguish purchases, installment occurrences, refunds, card payments, balance carryover, interest, fees/taxes, rewards and transfers.

### Your task

Review all supplied planning documents as if implementation will begin immediately after your review.

Do not praise the plan. Find what is wrong, missing, ambiguous, unnecessarily complex, risky or internally inconsistent.

For every finding, classify severity:

- BLOCKER — likely to cause incorrect money, security failure, unusable UX, or major rewrite;
- IMPORTANT — should be resolved before or during MVP;
- NICE_TO_HAVE — valuable but can wait.

For each finding provide:

1. ID;
2. severity;
3. area;
4. exact problem;
5. concrete failure scenario;
6. recommended change;
7. which document/section should be changed;
8. whether it changes MVP scope.

### Specifically audit these areas

#### A. User experience
- Does the product still require too much bookkeeping?
- Are any flows too complex for mobile?
- Is the “pending inbox” sufficient?
- Are there hidden setup burdens?
- Are there redundant screens/features?
- Can important decisions be reduced to fewer taps?
- Is “economic spending vs debt/cash view” understandable to a non-accountant?

#### B. Credit-card semantics
Search aggressively for missing cases:
- installment plans;
- original purchase not available in historical statements;
- deferred installments;
- early installment closure;
- partial refund;
- installment refund;
- cancelled/reversed authorization;
- statement credits;
- supplementary cards;
- cash advance;
- foreign currency;
- exchange-rate difference;
- interest/tax;
- minimum payment;
- overpayment/credit balance;
- card annual fee;
- statement date vs transaction/posting date;
- duplicate merchant/amount on same day.

Do not assume Turkish bank statements are simple.

#### C. Reconciliation
- Is source-event → transaction modeling sufficient?
- Can e-mail and statement matching safely avoid duplicates?
- What happens when e-mail amount differs from posted amount?
- What happens with tips, FX, reversals or offline terminal completion?
- Are match confidence and user review rules specified enough?
- Can reconciliation corrupt prior user corrections?

#### D. Rules / classification
- Could rules overgeneralize?
- Are contextual rules understandable?
- Does rule precedence create surprising behavior?
- Is AI confidence meaningful enough?
- How should rules change when merchants use payment processors?

#### E. AI / ChatGPT tools
- Are tools too broad?
- Are important tools missing?
- Are file imports actually feasible in the intended integration architecture?
- Could prompt injection in bank text trigger write actions?
- Are bulk writes safe?
- Is undo robust?
- Is there a dangerous mismatch between plugin permissions and application permissions?
- Which operations should never be exposed to the model?

#### F. Security/privacy
- RLS design gaps;
- OAuth scope;
- file retention;
- audit privacy;
- raw e-mail handling;
- backups;
- public repository risk;
- secrets;
- signed URLs;
- account deletion/export;
- prompt injection;
- cross-household access.

#### G. Architecture
- Is Supabase/Next.js/MCP overkill or underpowered?
- Are there too many services for one household?
- Which components can be collapsed?
- Which abstractions are premature?
- Which abstractions are essential before code?
- Is source_event separate from transaction correct?
- Is an event-sourcing model accidentally being reinvented?

#### H. Testing
Find missing financial invariants and edge cases.
What bugs would still pass the proposed tests?

#### I. Scope/time
The user explicitly does not want this project to consume excessive time.

Identify:
- what should be removed from MVP;
- what should be postponed;
- what can be borrowed from an existing open-source project;
- what should be built custom because the semantics are specific;
- the smallest vertical slice that proves the product.

### Required output

Start with:

## Executive verdict

Give:
- GO;
- GO WITH CHANGES;
- or STOP / REDESIGN.

Then:

## Top 10 risks

Ranked.

Then:

## Findings

A table/sections grouped by severity.

Then:

## Missing user journeys

List concrete flows absent from the plan.

Then:

## Missing financial edge cases

Be exhaustive.

Then:

## Simplification opportunities

Specifically tell us what NOT to build.

Then:

## Proposed revised MVP

Give the smallest version that still proves:
- statement correctness;
- low-effort UX;
- ChatGPT read/write usefulness.

Then:

## Architecture changes

Only changes you genuinely recommend.

Then:

## Questions that must be answered before coding

Maximum 15, ranked.

Finally:

## Patch list

Provide a document-by-document patch list. Do not rewrite all documents; identify the exact changes that should be made.

Be adversarial and concrete. Assume incorrect financial semantics are unacceptable and user time is scarce.
