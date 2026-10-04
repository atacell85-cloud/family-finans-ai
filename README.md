# Family Finans AI

AI-first personal and household finance system.

The product goal is simple: **the user should not have to do bookkeeping**. Financial activity should arrive automatically where possible, the system should classify what it knows, ask only about uncertainty, reconcile statements, and remain controllable from both a mobile-first PWA and ChatGPT.

## Product promise

- Import bank e-mails and card statements.
- Reconcile e-mail events with posted statement transactions.
- Distinguish real spending from debt movement, payments, installments, refunds, interest, tax and rewards.
- Track who used the payment instrument separately from who/what the spending belongs to.
- Classify spending as required, necessary/flexible, discretionary, or unknown.
- Preserve persistent user rules across models and conversations.
- Allow ChatGPT to read, add, edit, classify, split and reconcile financial data through a controlled tool layer.
- Maintain audit history and undo for AI-initiated changes.
- Optimize for **less than 10 minutes of manual cleanup per month**.

## Current status

Planning / architecture phase. No production code yet.

## Start here

1. [Master Plan](docs/MASTER_PLAN.md)
2. [Product Principles](docs/PRODUCT_PRINCIPLES.md)
3. [UX Specification](docs/UX_SPEC.md)
4. [User Journeys](docs/USER_JOURNEYS.md)
5. [Architecture](docs/ARCHITECTURE.md)
6. [Data Ingestion](docs/DATA_INGESTION.md)
7. [Statement Reconciliation](docs/STATEMENT_RECONCILIATION.md)
8. [Classification Rules](docs/CLASSIFICATION_RULES.md)
9. [Credit Card Model](docs/CREDIT_CARD_MODEL.md)
10. [ChatGPT Plugin / MCP](docs/CHATGPT_PLUGIN_MCP.md)
11. [Security & Privacy](docs/SECURITY_PRIVACY.md)
12. [Test Strategy](docs/TEST_STRATEGY.md)
13. [Roadmap](docs/ROADMAP.md)
14. [Acceptance Criteria](docs/ACCEPTANCE_CRITERIA.md)
15. [Decisions](docs/DECISIONS.md)
16. [Open Questions](docs/OPEN_QUESTIONS.md)
17. [Claude Review Prompt](docs/CLAUDE_REVIEW_PROMPT.md)

## Scope boundary

This is **not** intended to become a general accounting suite, brokerage dashboard, payment app, bank credential scraper, or investment platform. The first objective is reliable household spending intelligence with extremely low user effort.

## Privacy note

Do not commit real bank statements, card numbers, e-mail bodies, credentials, tokens, or personally identifying financial data to this repository. Test fixtures must be synthetic or irreversibly anonymized.
