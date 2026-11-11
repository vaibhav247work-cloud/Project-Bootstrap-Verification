# JSON and Scale-Ready Database Design

## Goal

Design core data so it remains correct and performant from a small launch to large usage, while keeping evolving configuration flexible. Do not promise that a schema will never change; require safe, backward-compatible evolution instead.

## The storage decision

| Data shape/use | Correct default |
|---|---|
| Identity, tenant, ownership, status, money, dates, capacity, relationships | Normal relational columns/tables |
| Repeated items or many-to-many records | Child/junction table |
| Frequently filtered, sorted, joined, unique, range-queried, or permission-sensitive value | Normal column with appropriate constraint/index |
| Optional, nested, versioned, client-configurable rule details | JSON definition |
| Frequently queried scalar inside JSON | Promote to a normal column, or use a generated/indexed column after query-plan review |

## JSON rules

1. **MUST NOT** use JSON merely to avoid designing relational data.
2. **MAY** use JSON for genuinely flexible configuration, optional metadata, or a versioned declarative rule definition.
3. **MUST** validate JSON structure, types, allowed fields, allowed operators, and version at the backend boundary.
4. **MUST NOT** store or execute arbitrary client-provided JavaScript, SQL, expressions, or code from a JSON field.
5. **MUST** keep security/authorization decisions, tenant ownership, money, statuses, date ranges, capacity, and relationship keys in relational fields.
6. **MUST** document query/index strategy before filtering, sorting, joining, or reporting on a JSON path at meaningful scale.
7. **MUST** version configurable JSON rules so old records can be interpreted after future rule changes.
8. **SHOULD** preserve the exact rule-set version or calculated result used for a completed financial/booking/approval transaction when historical reproducibility matters.

## Safe rule-definition example

```text
booking_rule_sets
  id, organization_id, code, status, version,
  effective_from, effective_to, definition_json,
  created_at, updated_at, created_by_user_id, updated_by_user_id
```

Example `definition_json`:

```json
{
  "minimum_notice_hours": 24,
  "maximum_days_ahead": 60,
  "allow_weekend_booking": false,
  "cancellation_policy": {
    "allowed_before_hours": 12,
    "refund_percent": 50
  }
}
```

The booking itself still uses relational fields such as `organization_id`, `resource_id`, `customer_id`, `start_at`, `end_at`, `status`, and `total_amount`. Availability, collision checks, reporting, and tenant isolation must never depend only on opaque JSON.

## JSON query and indexing rule

MySQL JSON values are not directly indexed as ordinary columns. If a JSON value becomes a real query path, first measure it. Then either:

1. promote it to a normal relational column; or
2. extract a stable scalar into a generated column and index that column.

Use `EXPLAIN`/query-plan review on representative data. Do not create JSON-path indexes speculatively.

## Scale-ready foundations

1. **MUST** use `BIGINT` identifiers for high-growth transactional tables unless an ADR selects another key strategy.
2. **MUST** use pagination/cursor patterns for collections and bounded batches for imports/exports.
3. **MUST** keep transaction rows narrow; put large documents/files in object storage and reference them from the database.
4. **MUST** separate hot operational tables from high-volume audit/event/history data and define archive/retention.
5. **MUST** use background jobs for long-running exports, imports, notifications, and recalculations.
6. **MUST** use measured query plans, load tests, and monitoring before choosing cache, read replica, denormalized read model, or partitioning.
7. **MUST NOT** add partitioning, sharding, read replicas, or cache as a default architectural decoration. Each requires an ADR and operational plan.
8. **MUST** plan schema changes as additive/backward-compatible migrations: add, deploy compatible code, backfill, switch reads, then remove later.

## 1M-scale readiness definition

"Ready for growth" means the product has correct keys, constraints, tenant isolation, query boundaries, observability, archive strategy, and safe migrations. It does not mean one early schema can predict every future product requirement without change.
