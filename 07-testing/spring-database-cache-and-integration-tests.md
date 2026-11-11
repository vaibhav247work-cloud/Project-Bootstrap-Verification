# Spring Boot, MySQL, Redis and Integration Testing Standard

## Default stack

Use JUnit 5 with Spring Boot test support. Use **Testcontainers** for integration tests that depend on MySQL, Redis, message broker, object storage emulator or another owned runtime dependency. Unit tests remain fast and do not start the full application context.

```text
unit test                 -> no Spring / no real infrastructure
Spring slice/module test  -> selected Spring boundary where useful
integration test          -> Spring Boot + real containerized dependencies
E2E                       -> separately deployed full product stack
```

Do not claim MySQL/Redis compatibility from H2, an in-memory map, mocked repository, or a local manually configured database.

## Spring test boundaries

| Test kind | Use for | Do not use for |
| --- | --- | --- |
| Plain JUnit | domain rules, mappers, validation, policies | JPA/Spring configuration behaviour |
| Web/controller slice | JSON binding, request validation, HTTP status/error contract | final authorization/transaction/database confidence alone |
| JPA/repository integration | SQL, mappings, queries, locks, paging, constraints | replacing a business-workflow test |
| Full Spring integration | security filters, transactions, datasource routing, cache/outbox/job wiring | every tiny calculation |
| Deployed E2E | critical cross-stack journey | exhaustive edge-case testing |

- Tests load the smallest context that proves the intended contract.
- Spring configuration is explicit for test profile; no test silently uses developer/prod credentials or endpoint configuration.
- Constructors/dependencies remain production-like. Do not add test-only public hooks that weaken application design; inject clocks/IDs/ports/interfaces where deterministic control is legitimate.
- Fixed time, random IDs, queues, remote calls and async executors are controllable in tests.

## MySQL and migrations

1. Every versioned migration runs against an empty MySQL container and a representative upgraded database state.
2. Verify schema, constraints, indexes, generated columns, defaults, charset/collation, precision, foreign keys and required audit columns where applicable.
3. Migration tests prove safe forward upgrade. A database migration is generally forward-only; rollback is an operational recovery plan, not assumed automatic reversal.
4. Test query behaviour using real MySQL: pagination/cursor semantics, case/locale assumptions, JSON query/index behaviour, time-zone conversion, uniqueness and `NULL` behaviour.
5. Test concurrent conflict scenarios: optimistic lock/version conflict, unique race, booking/inventory contention, transaction isolation and retry/idempotency behaviour.
6. Seed the smallest data set that demonstrates the condition. Large-scale query/load tests are separate performance tests, not normal unit/integration fixtures.
7. Never use production data without approved anonymisation, minimisation, access control, retention and legal/privacy approval.

## Database-per-tenant verification

Database-per-tenant routing is high risk and requires tests with at least two separate tenant databases:

```text
tenant A request -> datasource A + cache namespace A + tenant A data
tenant B request -> datasource B + cache namespace B + tenant B data
wrong/missing context -> denied/safe failure; never fallback to another tenant
```

- Verify request/session-derived trusted tenant context, dynamic datasource lookup, lazy pool creation/eviction and safe cleanup.
- Confirm no query, native SQL, job, cache key, file metadata, audit event or external callback crosses tenant boundary.
- Confirm an equal record ID in two tenant databases cannot return/cache the wrong record.
- Test tenant switch/logout/session revoke invalidates tenant-specific request/query/cache context.
- Test a failed/unavailable tenant datasource fails only the affected request/tenant according to policy and does not corrupt/close active pools for another tenant.

## Redis and cache/session tests

Use real Redis container tests for TTL, serialization, namespace, invalidation and outage fallback behaviour.

- Assert key format includes tenant when Redis is shared: `app:tenant:<tenant-code>:...`.
- Verify TTL/idle refresh follows server-calculated session expiry—not a client-supplied value.
- Test cache-aside miss/hit, after-commit invalidation, stampede protection where implemented, serialization/schema compatibility and bounded key/value size.
- Role, permission, membership, temporary override, account status, password/reset and session revoke changes evict effective-access/session cache promptly.
- Redis failure is cache miss/controlled failure; it never grants authorization or bypasses MySQL source-of-truth checks.
- Tests must not assert a particular implementation key unless that key is an intentional documented contract; assert isolation/expiry/invalidation outcome.

## Transactions, outbox, jobs and events

- Test that business data, audit event and outbox row commit atomically, and no external event is delivered when transaction rolls back.
- Worker tests cover duplicate delivery, idempotent consumer behaviour, retry/backoff, poison/dead-letter handling, ordering assumptions, observability and manual recovery.
- Scheduled jobs test lock/leader coordination so parallel nodes do not perform a supposed single job twice.
- Async work propagates only approved minimal correlation/tenant/security context; threads never inherit stale context from an earlier request.
- Use controllable clock and executor in tests. No sleep-based timing assertion.

## Third-party HTTP/integration tests

For each provider, maintain version-controlled fixtures for valid, invalid, slow, timeout, 429, 5xx, malformed and schema-changed responses.

- Test request contract: method, URL path/query, headers, authentication/signature without exposing secrets, timeout and idempotency key where required.
- Test response mapping, validation, retry eligibility, circuit-breaker/fallback, rate-limit handling, audit/log redaction and user-visible outcome.
- Contract tests run against approved sandbox/provider contract when available; normal CI does not call uncontrolled external services.
- Webhook tests validate signature, replay/duplicate protection, source allow-list where applicable, payload schema, tenant/customer association and idempotent processing.
- File/object-store tests cover authorization, signed URL expiry, upload/download metadata and cleanup/lifecycle without real customer files.

## Required CI grouping

| Group | Runs when | Expectation |
| --- | --- | --- |
| Unit/slice | every change | fast, parallel, deterministic |
| MySQL/Redis integration | backend/db/cache/security change and main branch | containerized, migration-ready |
| Full integration/outbox/job | affected module and release branch | realistic async/transaction failure paths |
| Provider contract/sandbox | scheduled/release or provider change | isolated credentials/environment, retained report |
| E2E | critical path/release | deployed stack and controlled test data |

Failed tests are fixed at the underlying application/test-environment cause. Do not increase retries or disable a test merely to make CI green.
