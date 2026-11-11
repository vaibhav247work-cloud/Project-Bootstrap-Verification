# MySQL Database Testing Standard

## Required test layers

| Test | What it proves |
|---|---|
| Migration test | Empty and upgrade-path databases can reach the target schema |
| Constraint test | PK, FK, unique, check, and null rules prevent invalid data |
| Repository/integration test | Queries, transactions, tenant filters, and mappings work against MySQL |
| Concurrency test | Critical simultaneous actions do not duplicate or lose data |
| Performance test | Important queries/reports meet agreed targets on representative data |
| Backup restore drill | Recovery procedure actually restores usable data |

## Mandatory rules

1. **MUST** run migrations against a fresh database and a representative previous-version schema in CI or the release pipeline.
2. **MUST** test critical database behaviour against MySQL, not only an in-memory replacement.
3. **MUST** test tenant isolation with at least two organizations for every tenant-scoped repository/API flow.
4. **MUST** test business uniqueness, status transitions, soft-delete behaviour, and authorization-related data constraints.
5. **MUST** test failure/rollback for multi-step transactional operations.
6. **MUST** test the actual backup restore process periodically and record the result, duration, and recovery gap.
7. **SHOULD** use masked/synthetic representative data for performance tests; never use uncontrolled production PII.

## Release gate

A migration or data-sensitive feature cannot be released until its migration test, relevant constraints/integration tests, and rollback/recovery plan pass.
