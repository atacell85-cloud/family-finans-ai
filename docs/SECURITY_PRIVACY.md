# Security & Privacy

## 1. Threat model

This system contains sensitive household financial information.

Primary risks:

- unauthorized account access;
- cross-household data leakage;
- leaked statement files;
- leaked e-mail content;
- exposed service keys;
- AI tool misuse;
- overly broad OAuth scopes;
- prompt-driven destructive mutation;
- sensitive logs;
- public repository accidents.

Security is an MVP requirement.

---

## 2. Data minimization

Collect only what is needed.

Never store:
- CVV;
- bank password;
- internet banking credentials;
- full authentication cookies;
- full card PAN unless absolutely required (it is not expected to be).

Prefer:
- card last four digits;
- bank/issuer;
- card nickname;
- stable internal ID.

---

## 3. Repository policy

No production financial data in Git.

Forbidden:
- real statements;
- raw bank e-mails;
- tokens/API keys;
- personal addresses;
- account numbers;
- full names in fixtures if avoidable;
- screenshots containing real financial data.

Fixtures must be synthetic or irreversibly anonymized.

If the repository is public, assume every committed byte is permanently public even after deletion.

---

## 4. Authentication

Recommended:
- Supabase Auth;
- magic link/passkey or strong supported login;
- short-lived sessions;
- secure cookie/session handling.

Avoid custom password storage.

---

## 5. Authorization / RLS

RLS is mandatory.

Every data-bearing row should be scoped to household directly or through a verifiable relation.

Policies must be tested for:
- valid household member read;
- valid household member write;
- cross-household read denied;
- cross-household write denied;
- anonymous access denied.

Service role key must only exist in trusted backend runtime.

---

## 6. MCP authorization

The MCP/plugin session must bind to an authenticated user.

Rules:
- resolve household server-side;
- never accept model-provided household ID as authority;
- validate every referenced transaction/import/rule;
- enforce per-tool permission class.

---

## 7. OAuth/mail access

Use the minimum scope needed.

Prefer:
- read-only Gmail access;
- query filtering for bank sender/subject;
- no contacts/calendar scopes;
- no sending mail.

Do not store OAuth refresh tokens in client-side storage.

---

## 8. Statement files

Use private object storage.

Requirements:
- no public bucket;
- signed short-lived URLs;
- access check before issuing URL;
- malware/content-type checks where appropriate;
- file size limits;
- retention policy.

Default option should allow deleting raw files after successful parse/reconciliation while retaining normalized evidence.

---

## 9. Raw e-mail content

Default:
- retain provider message ID;
- retain extracted structured values;
- avoid retaining full body indefinitely.

If raw body is retained for debugging:
- encrypt at rest via platform;
- strict access;
- short TTL;
- no inclusion in general logs.

---

## 10. Encryption

Use:
- TLS in transit;
- platform encryption at rest;
- secret manager for credentials.

Do not invent custom cryptography unless a clear field-level encryption requirement arises.

---

## 11. Secrets

Secrets include:
- Supabase service role key;
- OAuth client secret;
- model provider key;
- MCP signing/auth secrets.

Rules:
- environment/secret store only;
- never `.env` committed;
- rotate if exposed;
- staging and production separate.

---

## 12. Logging

Logs may include:
- request ID;
- import ID;
- parser version;
- error code;
- counts;
- masked payment instrument.

Logs must not include:
- raw e-mail body;
- statement full text;
- access token;
- full account/card number;
- private attachment URL with long expiry.

---

## 13. AI privacy

Send the minimum context required for a model task.

For merchant classification, do not send:
- full statement;
- unrelated transactions;
- household identity details;

if merchant/amount/context subset is enough.

AI provider requests should be isolated behind an adapter so policy can change later.

---

## 14. AI write controls

AI has no unrestricted database access.

Controls:
- whitelisted tools;
- strict schemas;
- server-side authorization;
- idempotency;
- per-tool risk class;
- audit;
- undo;
- confirmation for broad/high-impact writes.

Prompt injection inside statement/e-mail text must be treated as untrusted data, never as instructions to the tool layer.

---

## 15. File-content prompt injection

A statement or e-mail may contain arbitrary text.

Parser/AI extraction prompts must clearly separate:
- untrusted source content;
- system extraction instructions.

The model must not be allowed to invoke tools based on instructions embedded in financial documents.

Document text is data, not authority.

---

## 16. Rate limiting and abuse

Apply:
- per-user API rate limits;
- upload limits;
- MCP write rate limits if needed;
- worker concurrency limits;
- retry caps.

This protects both cost and integrity.

---

## 17. Backup security

Backups contain sensitive data.

Requirements:
- restricted access;
- encryption;
- retention policy;
- restore test;
- delete/export policy understood.

---

## 18. User data rights

User should be able to:
- export structured data;
- inspect rules;
- inspect audit changes;
- delete account/data according to policy.

Do not create hidden model-only state that cannot be exported or explained.

---

## 19. Security review checklist

Before production:
- RLS tests pass;
- secrets scan passes;
- dependency vulnerability scan;
- file upload validation;
- cross-household tests;
- MCP authorization tests;
- destructive tool review;
- audit completeness;
- backup restore test;
- data export test;
- privacy retention settings documented.

---

## 20. Incident response

Document:
1. how to revoke credentials;
2. how to disable MCP writes;
3. how to stop e-mail ingestion;
4. how to rotate keys;
5. how to inspect audit logs;
6. how to restore from backup.

Feature flags should permit disabling risky integrations without taking the core app offline.
