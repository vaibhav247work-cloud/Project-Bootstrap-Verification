# Data Model and ERD Template

> Use one document per bounded domain/module. It is a design aid, not a substitute for reviewed migrations. Apply the MySQL standards in [`03-databases/mysql/`](../03-databases/mysql/) to all final schema decisions.

## 1. Scope and ownership

| Field | Value |
| --- | --- |
| Domain/module | `[name]` |
| PRD/workflows | `[links]` |
| Tenant database | `[tenant database rule/name pattern]` |
| Data owner | `[role/team]` |
| Classification | `[internal/personal/sensitive]` |
| Migration owner | `[role/team]` |

## 2. ERD

Embed or link the ERD source (for example DBML, Mermaid, diagram source) and rendered diagram. Show primary keys, foreign keys, cardinality, uniqueness and lifecycle ownership.

`[ERD link or diagram]`

## 3. Entity catalogue

| Entity/table | Purpose | Primary key | Important fields and constraints | Relationships | Lifecycle/audit | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| `[plural_snake_case]` | `[purpose]` | `[id]` | `[unique/check/not null]` | `[FK/cardinality]` | `[created/updated/deletion rule]` | `[notes]` |

For each entity, state whether `status` is a true lifecycle state and whether `is_active` is independently needed. Do not add either mechanically.

## 4. Access and query design

| Use case | Read/write actors | Tenant/permission rule | Expected volume | Query/filter/sort | Required index | Pagination/export rule |
| --- | --- | --- | --- | --- | --- | --- |
| `[use case]` | `[roles]` | `[rule]` | `[estimate]` | `[query shape]` | `[index]` | `[bounded/cursor/etc.]` |

Indexes follow actual query patterns and are verified with representative `EXPLAIN`; do not index every column by habit.

## 5. Integrity, concurrency and flexible data

- Invariants enforced by database/application: `[answer]`
- Foreign-key/delete behaviour: `[restrict/soft-delete/etc.]`
- Transaction and locking/versioning rule: `[answer]`
- Idempotency/deduplication key: `[answer]`
- JSON use: `[validated, versioned configuration only; indexed query fields remain columns]`
- Retention/archive/purge need: `[answer]`
- Audit events and actor attribution: `[answer]`

## 6. Migration and recovery plan

| Change | Backward-compatible step | Backfill/validation | Rollback or forward-fix | Tenant sequencing | Evidence |
| --- | --- | --- | --- | --- | --- |
| `[change]` | `[step]` | `[plan]` | `[plan]` | `[plan]` | `[test/restore proof]` |

## 7. Review checklist

- [ ] Names, keys, constraints and relationships follow the database standard.
- [ ] Tenant isolation is preserved; no unapproved cross-tenant join/path exists.
- [ ] Sensitive data is minimized and access/audit needs are stated.
- [ ] Query patterns, pagination and indexes are designed for projected scale.
- [ ] Migration, backup/recovery and test implications are known.
