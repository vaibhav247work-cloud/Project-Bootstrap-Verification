# MySQL Schema Rules

## Naming

1. **MUST** use lowercase `snake_case` for tables, columns, indexes, and constraints.
2. **MUST** use plural table names: `users`, `booking_items`, `audit_logs`.
3. **MUST** use a meaningful singular foreign-key column name: `user_id`, `organization_id`; never vague names such as `ref_id`.
4. **MUST** use `created_at`, `updated_at`, and where applicable `deleted_at` consistently.
5. **MUST** use `created_by_user_id` and `updated_by_user_id` for a user-managed transactional record. A service/system actor must use a documented system principal rather than a misleading human user ID.
6. **SHOULD** name constraints and indexes predictably: `pk_<table>`, `fk_<child>__<parent>`, `uq_<table>__<columns>`, `idx_<table>__<columns>`.

## Identity and types

1. **MUST** give every persistent business table a primary key.
2. **SHOULD** use `BIGINT UNSIGNED` generated IDs as the default internal primary key. Use UUID/ULID only when external exposure, distributed generation, or migration requirements justify it; document that decision.
3. **MUST** use the exact same compatible type, size, and unsigned/signed setting on both sides of a foreign-key relation.
4. **MUST** use `DECIMAL(precision, scale)` for currency and exact quantities; never `FLOAT`/`DOUBLE` for money.
5. **MUST** use `DATETIME` or `TIMESTAMP` consistently according to the project convention. Store instants in UTC.
6. **MUST** use `NOT NULL` whenever absence is not meaningful. Do not use `NULL` as a substitute for empty string, zero, or an unknown design decision.
7. **MUST** use database `CHECK`, `UNIQUE`, `NOT NULL`, and foreign-key constraints for stable data invariants where supported and practical.
8. **SHOULD** store controlled values in reference tables or carefully managed enums. Do not create uncontrolled free-text status fields.

## Relationships and integrity

1. **MUST** add foreign keys for real parent-child relationships unless there is a documented performance, sharding, legacy, or lifecycle exception.
2. **MUST** explicitly choose `ON DELETE` and `ON UPDATE` behaviour. Default to `RESTRICT`; use `CASCADE` only when child records have no independent value.
3. **MUST NOT** cascade delete financial, audit, approval, identity, or legally retained records.
4. **MUST** resolve many-to-many relations with a junction table (for example `user_roles`) and add a unique constraint on the relationship pair unless duplicates are meaningful.
5. **MUST** keep tenant/organization ownership explicit on every tenant-scoped table. Do not rely only on an application-level join for isolation.

## Standard columns

### Normal transactional table

```sql
id              BIGINT UNSIGNED NOT NULL AUTO_INCREMENT
organization_id BIGINT UNSIGNED NULL -- required for tenant-scoped records
created_at      DATETIME NOT NULL
updated_at      DATETIME NOT NULL
created_by_user_id BIGINT UNSIGNED NOT NULL
updated_by_user_id BIGINT UNSIGNED NOT NULL
```

### Add only when needed

- `deleted_at`: soft deletion is a defined product requirement.
- `version`: optimistic locking is needed for concurrent editing.
- `status`: the entity has a controlled lifecycle.

`created_by_user_id` and `updated_by_user_id` are mandatory for user-managed transactional records (for example bookings, employee records, offers, approvals, and settings). They are not required for pure reference/catalog tables, immutable integration-event records, aggregate/reporting tables, or technical join tables unless the project needs attribution there too.

For background jobs, imports, or system actions, set these fields to a documented non-human system user/principal. Do not leave them blank or pretend a real user made the change.

These fields show only the latest actor. They never replace an append-only audit log when history, compliance, dispute resolution, or sensitive change tracking is required.

## Status and active state

1. **MUST** use a controlled `status` field when an entity has a lifecycle, for example `draft`, `pending_approval`, `active`, `suspended`, `expired`, `cancelled`, or `archived`.
2. **MUST** define every allowed status, permitted transition, the actor allowed to make it, and its business effect in the module specification.
3. **MUST NOT** use free-text status values. Use an application enum plus database validation/reference data according to the migration strategy.
4. **MAY** use `is_active` only when it represents one independent on/off capability and is not merely a duplicate of `status`.
5. **MUST** document the distinction if both exist. Example: an offer can have `status = approved` but `is_active = false` while temporarily disabled; an expired offer is `status = expired` and cannot become active without a new business transition.
6. **SHOULD** use a single `status` field instead of `is_active` where there is more than a simple two-state lifecycle.

For a user account, a practical default is `status` such as `invited`, `active`, `suspended`, `disabled`. Do not maintain both `user_status` and `is_active` unless their meanings are explicitly different.

## Soft delete

1. **MUST** decide deletion behaviour per entity; do not add `deleted_at` blindly.
2. **MUST** ensure normal reads exclude soft-deleted records.
3. **MUST** design uniqueness with deletion in mind; otherwise a deleted email/code may block a legitimate future record.
4. **SHOULD** use archive/anonymization rather than soft delete for data with retention or privacy rules.

## Example: users and roles

```text
users                 user_roles                 roles
-----                 ----------                 -----
id PK           1--*  user_id FK           *--1  id PK
organization_id       role_id FK                 code UQ
email                 UQ(user_id, role_id)       name
password_hash
```

Passwords are stored only as a secure hash. Login/session/refresh-token data belongs in separate tables with its own expiry, revocation, and audit policy.
