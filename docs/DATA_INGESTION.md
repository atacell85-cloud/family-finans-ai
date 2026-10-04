# Data Ingestion

## 1. Goal

Ingest financial activity with minimal user effort while preserving provenance and preventing duplication.

Supported source classes:

- bank e-mail notification;
- PDF statement;
- CSV/XLSX export;
- manual entry;
- ChatGPT/tool entry.

---

## 2. Common ingestion contract

Every source should first become a **source event**, not a final transaction.

Minimum normalized fields:

- source_type;
- source_provider;
- external_id/idempotency_key;
- occurred_at;
- posted_at if known;
- raw_description;
- amount;
- currency;
- direction;
- masked account/card identity;
- merchant candidate;
- event type candidate;
- parser name/version;
- confidence;
- source metadata.

Only after normalization does the domain layer create/match/reconcile a transaction.

---

## 3. E-mail ingestion

### 3.1 Purpose

Near-real-time provisional activity.

### 3.2 Access

Use least-privilege mailbox access.

Prefer:
- search restricted to known bank senders/subjects;
- no broad long-term e-mail mirroring;
- provider message ID retained;
- raw body retention disabled or short-lived by default.

### 3.3 Pipeline

1. discover new matching message;
2. verify sender/domain pattern;
3. check message ID idempotency;
4. choose bank e-mail parser;
5. extract normalized candidate;
6. create source event;
7. merchant normalization;
8. rule engine;
9. create provisional transaction or pending review;
10. mark processing state.

### 3.4 Failure states

- unsupported template;
- malformed amount;
- ambiguous card;
- unknown currency;
- message duplicate;
- sender not trusted.

Failures must be visible in an ingestion error queue.

---

## 4. PDF statement ingestion

### 4.1 Purpose

- historical import;
- authoritative monthly reconciliation.

### 4.2 File lifecycle

1. upload to private temporary storage;
2. calculate file hash;
3. deduplicate prior import;
4. detect bank/template;
5. parse;
6. validate arithmetic;
7. build preview;
8. user/AI reviews exceptions;
9. commit;
10. retain/delete raw file according to policy.

### 4.3 Parsing levels

1. native PDF text;
2. deterministic bank adapter;
3. layout/table extraction;
4. vision/OCR fallback;
5. AI extraction fallback.

Do not start with OCR if native text is available.

### 4.4 Parsed statement summary

Normalized statement metadata should include where available:

- issuer/bank;
- account/card identity;
- statement period;
- opening/previous balance;
- payments/credits;
- purchases/charges;
- interest;
- fees;
- taxes;
- closing balance;
- minimum payment;
- due date;
- card-level subtotals;
- statement currency.

### 4.5 Parsed row fields

- transaction date;
- posting date;
- description;
- amount;
- sign/direction;
- transaction type candidate;
- installment number/count;
- remaining installment hint;
- reward/points where useful;
- card sub-account/cardholder if present;
- bank-specific row ID/reference;
- raw row text/position reference.

---

## 5. CSV/XLSX ingestion

Use explicit mapping profiles.

Flow:

1. detect known bank/profile;
2. if unknown, ask user to map columns once;
3. save mapping profile;
4. normalize rows;
5. use same transaction/reconciliation pipeline as PDF.

Avoid maintaining a completely separate import path.

---

## 6. Manual entry

Manual entry creates a source event with source_type=manual.

Natural-language input should be parsed into a preview before commit when ambiguity exists.

Minimum required business data:

- amount;
- currency;
- date/time;
- direction/type;
- account/payment method.

Category/owner/necessity may remain unknown and enter review.

---

## 7. ChatGPT entry

ChatGPT calls the same domain command used by manual UI.

Required protections:

- authenticated household scope;
- client/request idempotency key;
- audit actor=chatgpt;
- no arbitrary database field access;
- validated enum/range schemas;
- explicit bulk preview for wide changes.

---

## 8. Merchant normalization

Normalization is an ingestion-adjacent service.

Steps:

1. trim/normalize case/Unicode;
2. remove known processor noise cautiously;
3. resolve exact alias;
4. resolve bank-specific alias;
5. fuzzy candidate only if safe;
6. canonical merchant or unknown.

Do not over-normalize generic processor names such as IYZICO/MONEYPAY without preserving downstream merchant text.

---

## 9. Ingestion confidence

Confidence is evidence-specific.

Example fields:

- parse_confidence;
- merchant_confidence;
- type_confidence;
- classification_confidence.

Do not collapse everything into one opaque score if different decisions need separate review.

---

## 10. Idempotency

### E-mail
Unique on provider + message_id.

### File
Unique hash can flag re-upload. Do not assume identical hash for every equivalent statement export.

### Statement row
Use statement identity + deterministic row fingerprint where available.

### Manual/ChatGPT
Use request_id to prevent retry duplication.

---

## 11. Raw-source retention

Default privacy posture:

### E-mail
Store message ID and extracted structured fields. Raw body only if necessary for parser troubleshooting and with short retention.

### Statement
Store raw file privately only as long as the user/policy requires.

### Parser diagnostics
Prefer structural metadata and row references over full raw content in logs.

---

## 12. Backfill

Historical backfill should be explicit.

Options:

- upload previous statements;
- import CSV history.

E-mail ingestion should not automatically scan years of mailbox history unless the user requests it.

---

## 13. Import status model

Suggested:

- uploaded;
- detected;
- parsing;
- parsed;
- validation_failed;
- preview_ready;
- needs_review;
- committed;
- partially_committed only if explicitly designed;
- failed;
- cancelled.

Prefer atomic commit of a statement import where practical.

---

## 14. Ingestion acceptance criteria

- reprocessing same source does not create duplicate expense;
- parser version/provenance stored;
- unsupported format fails safely;
- statement mismatch blocks silent commit;
- user sees concise preview rather than raw parser output;
- e-mail source can later reconcile to statement row;
- real sensitive source material never lands in Git or ordinary app logs.
