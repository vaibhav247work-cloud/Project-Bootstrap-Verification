# MySQL Performance and Scale Rules

## Principles

Measure real query paths and data shape before optimizing. Performance work must preserve correctness, tenant isolation, auditability, and recovery.

## Required rules

1. **MUST** define expected volume, growth, concurrent users, latency targets, and largest expected export/report for significant modules.
2. **MUST** inspect query plans for slow, large, or critical queries and retain index rationale where it is not obvious.
3. **MUST** paginate interactive lists; exports/imports/reports must use background jobs or bounded batches when large.
4. **MUST** avoid N+1 query patterns, unbounded reads, and loading large blobs into normal list endpoints.
5. **MUST** monitor slow queries, connection saturation, storage growth, lock/deadlock errors, backup duration, and database error rate.
6. **MUST** establish archive/retention handling for high-volume event, audit, notification, or integration tables.

## Scaling options

| Need | Consider | Prerequisite |
|---|---|---|
| Slow known query | Query/index improvement | Real query plan and representative data |
| Expensive repeat read | Cache/read model | Invalidation and staleness rule |
| Large export/report | Async job/read replica or reporting store | Access control and data freshness rule |
| Very large time-based table | Archive or partitioning | Retention/query pattern and operational plan |
| High read volume | Read replicas | Replication-lag tolerance and routing strategy |

Partitioning, sharding, replicas, caching, and denormalized tables require an ADR. They are not default optimizations.

## Data lifecycle at scale

Large append-only data must have a lifecycle: hot operational data, archived data, and deletion/anonymization when retention ends. The restore and reporting impact of archive choices must be tested before production use.
