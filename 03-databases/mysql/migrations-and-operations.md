# MySQL Migrations and Operations

## Migrations

1. **MUST** version every schema change in source control and apply it through the project migration tool.
2. **MUST** make each migration small, deterministic, and forward-only. Fix a bad released migration with a new migration; never edit migration history already deployed.
3. **MUST** test migrations on a production-like copy/shape before a risky release.
4. **MUST** plan data backfills separately from structural changes when tables are large.
5. **MUST** design backward-compatible changes for zero/low-downtime releases: add first, deploy compatible code, backfill, then remove later.
6. **MUST** document rollback/recovery for destructive or irreversible migrations.

## Backups and recovery

1. **MUST** automate production backups with defined retention and access control.
2. **MUST** encrypt backups at rest and in transit where the platform does not already guarantee it.
3. **MUST** test restoring a backup on a schedule and record the result.
4. **MUST** define acceptable Recovery Point Objective (RPO) and Recovery Time Objective (RTO) with the client/product owner.
5. **MUST NOT** use a backup policy that has never been restore-tested.

## Production access and observability

1. **MUST** use least-privilege database accounts: application runtime, migration, read-only/reporting, and administration should not share one broad account.
2. **MUST NOT** give the application account schema-administration privileges in normal production operation.
3. **MUST** monitor capacity, connections, slow queries, errors, backup success, and replication/availability status where applicable.
4. **SHOULD** enable and review slow-query information in production according to platform capacity and privacy policy.
