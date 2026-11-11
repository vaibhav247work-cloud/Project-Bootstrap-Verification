# MySQL Sensitive Data and Privacy Rules

This is an engineering baseline, not legal advice. The client and applicable jurisdictions determine legal basis, retention periods, data-subject rights, and contractual obligations.

## Data inventory and classification

Before storing a new sensitive field, document its data class, purpose, owner, access roles, encryption/masking need, retention, and deletion/anonymization path.

| Class | Examples | Minimum treatment |
|---|---|---|
| Public | public catalogue data | Integrity controls |
| Internal | internal operational metadata | Role-restricted access |
| Personal/PII | name, email, phone, employee identifier | Minimize, access control, retention decision |
| Sensitive/high-risk | financial, health, identity documents, biometric data | Need-to-know access, stronger audit, encryption/masking assessment |
| Secret | passwords, API keys, tokens | Do not store unless essential; hash or use a secrets system |

## Mandatory rules

1. **MUST** collect only fields required for a documented product purpose.
2. **MUST** classify personal/sensitive fields before implementation and include them in the project data inventory.
3. **MUST** restrict reads, writes, exports, and reports to the minimum roles and data scope required.
4. **MUST** use a strong one-way password hash; never store recoverable passwords.
5. **MUST** encrypt data in transit. Evaluate encryption at rest/field-level encryption for high-risk data according to threat model, platform controls, and client/regulatory requirements.
6. **MUST NOT** place sensitive values in logs, error messages, unrestricted audit snapshots, analytics events, URLs, or test fixtures.
7. **MUST** define retention plus deletion, archive, or anonymization behaviour for each high-risk record class.
8. **MUST** apply the same access and retention policy to exports, backups, replicas, and non-production data. Production PII must not be copied to local/dev environments without approved masking.
9. **SHOULD** pseudonymize/mask sensitive data in support, reporting, and lower environments where full identity is not required.

## Project gate

No feature storing sensitive data is ready for production until its data-inventory entry, access matrix, audit decision, retention decision, and test-data strategy are reviewed.
