# MySQL Indexing and Query Rules

Indexes make reads faster but make writes heavier. Create indexes for known query paths, not as decoration.

## Mandatory rules

1. **MUST** start with a primary key on every table.
2. **MUST** index a foreign-key column or use an index whose leading columns match that foreign-key column. MySQL requires indexes for foreign-key checks.
3. **MUST** add a unique index for every real business uniqueness rule, such as `organization_id + employee_code`.
4. **MUST** review the execution plan (`EXPLAIN`/`EXPLAIN ANALYZE` where available) for slow, large, or business-critical queries.
5. **MUST** decide indexes during project setup or feature design from real screen/API filters, sorting, expected data volume, tenant scope, and query plans. Do not prescribe an index only because a column exists.
6. **SHOULD** include tenant scope (often `organization_id`) in an index when the real query filters by tenant and the execution plan benefits from it.
7. **SHOULD** avoid standalone boolean/status indexes unless query volume and selectivity prove that they help; combine columns only for an actual query pattern.
8. **MUST NOT** add indexes to every column or use unbounded `SELECT *` in large/critical paths.

## Composite indexes

Order columns according to actual filters and sort pattern. A common tenant list query:

```sql
SELECT id, name, status
FROM bookings
WHERE organization_id = ? AND status = ?
ORDER BY created_at DESC
LIMIT 50;
```

Candidate index: `idx_bookings__organization_id_status_created_at`.

Verify with realistic production-like data. Do not copy an index blindly.

## Query safety

1. **MUST** use parameterized queries/ORM bindings. Never concatenate user input into SQL.
2. **MUST** paginate lists and exports; use stable, documented ordering.
3. **SHOULD** prefer keyset/cursor pagination for very large or frequently changing datasets.
4. **MUST** use transactions when several writes must either all succeed or all fail.
5. **MUST** keep transactions short; never perform slow external network calls inside a database transaction.
6. **SHOULD** avoid N+1 query patterns; measure and batch/join deliberately.

## JSON columns

1. **MAY** use `JSON` for genuinely variable, non-relational attributes.
2. **MUST NOT** use JSON to avoid modelling frequently filtered, joined, constrained, or permission-sensitive data.
3. **MUST** document schema expectations and indexing strategy for JSON fields.
