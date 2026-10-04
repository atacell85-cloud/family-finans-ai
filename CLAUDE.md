# CLAUDE.md

## Current project phase

This repository is in **planning review**. Do not implement application code yet.

## Your immediate task

Read:

- README.md
- every file under `docs/`
- especially `docs/CLAUDE_REVIEW_PROMPT.md`

Then perform the adversarial architecture/product/security/UX review exactly as requested in `docs/CLAUDE_REVIEW_PROMPT.md`.

## Output

Write the complete unedited review to:

`docs/CLAUDE_REVIEW.md`

Do not modify the master plan, architecture, roadmap, decisions, or any other planning document during this review.

The review is evidence for a later triage step; it is not permission to redesign the project directly.

## Important priorities

1. Financial correctness.
2. Very low user effort.
3. Mobile usability.
4. Safe ChatGPT read/write/import access.
5. Privacy/security.
6. Simplicity and low maintenance.
7. Avoiding overengineering.

Be particularly aggressive about finding:

- incorrect credit-card/installment semantics;
- duplicate/reconciliation failure modes;
- AI tool overreach;
- hidden UX bookkeeping;
- unnecessary infrastructure;
- missing edge cases;
- assumptions about ChatGPT/MCP capabilities that must be tested rather than assumed.
