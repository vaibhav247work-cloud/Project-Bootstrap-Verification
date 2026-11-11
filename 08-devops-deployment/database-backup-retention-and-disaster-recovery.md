# Database Migrations, Backups, Retention and Disaster Recovery Standard

## Scope and ownership

MySQL is the durable source of truth for business, identity, audit and refresh-session data. Every production database, including each tenant database, has a named operational owner, backup policy, migration state, retention policy, recovery target and tested restore procedure.

Redis is not a substitute for a database backup. Object storage/files, configuration, secrets, deployment manifests and critical external-provider state have their own backup/export/recovery plan; a database-only plan is incomplete.

## Migration operations

- Migrations are versioned, reviewed, deterministic and executed by one approved pipeline/job—not by every application instance during scale-out.
- Before applying a migration, verify target environment, tenant inventory, current schema version, free capacity, backup success, estimated lock/runtime and application compatibility window.
- Use expand/contract evolution: add compatible schema, deploy compatible code, backfill/verify, switch traffic, then later remove old structure. Do not combine irreversible drop/rename with a code rollout that has no safe rollback path.
- Large data backfills run in bounded, observable, resumable batches with rate/lock protection. They are not an unbounded startup task or single request.
- For database-per-tenant deployments, track migration state per tenant: tenant code, database identifier, target/current version, start/end, status, retry/recovery owner and error. Failures are visible and do not silently leave a tenant on incompatible schema.
- A migration success means version recorded plus application/read-write verification; it does not mean merely that the migration command exited zero.

## Backup policy

Each product documents exact RPO (maximum acceptable data loss) and RTO (maximum acceptable recovery duration), selected from client/business impact. Backup frequency, retention and replication design must demonstrate they meet these targets.

| Requirement | Standard |
| --- | --- |
| Scope | all tenant databases, platform/support DB if present, and required object/configuration data |
| Types | periodic full backup; incremental/binlog/point-in-time capability when RPO requires it |
| Protection | encrypted in transit/at rest, least-privilege backup/restore identities, immutable or protected retention where available |
| Separation | store copies outside the primary failure domain/account/region according to risk; do not rely on one disk/cluster |
| Integrity | verify job completion, size/content/checksum/restore readability; alert on missed/failed/outlier backup |
| Retention | documented daily/weekly/monthly/legal hold policy; purge only through controlled retention process |
| Access | audit backup listing/download/restore; never expose backup files to application/public network |

Enable and retain MySQL binary logs for the documented point-in-time recovery window when required. Point-in-time recovery restores a full backup and applies subsequent log changes to the selected safe point; it is planned/tested, not improvised during an incident.

## Backup execution rules

- Backups are automated, monitored and use application-consistent/database-consistent method suitable for MySQL deployment type. Do not copy live database files casually.
- Backup process must include schema, data, stored objects/configuration needed for restore and version metadata. For managed MySQL, document provider snapshots/log-retention semantics and restore limitations.
- A backup job has bounded resource/network impact, avoids unexpected production lock/latency, and reports duration, completion, bytes, source version, tenant scope and destination verification.
- Tenant databases may be backed up individually or through an approved fleet process; either way restore selection must identify exact tenant/database and prevent accidental overwrite of another tenant.
- Before major migration, upgrade or destructive repair, take/verify a current backup and test the recovery/forward-fix plan appropriate to blast radius.

## Restore and disaster recovery procedure

```text
declare incident -> stop/contain harmful writes -> identify tenant/system scope
-> select verified backup + recovery point -> restore into isolated target
-> validate schema/data/business invariants/tenant boundary -> controlled cutover
-> verify application/auth/jobs/monitoring -> communicate and preserve evidence
```

- Restore to an isolated target first unless an emergency runbook explicitly directs otherwise. Never overwrite a live production database based only on a guessed backup choice.
- Recovery runbook specifies authority, incident roles, customer/client communication, write freeze, credential/secrets implications, artifact/configuration versions, DNS/connection cutover, job/outbox handling and validation checklist.
- Validation includes migration version, tenant mapping, row/count/checksum or business-invariant checks, identity/session/security data, critical workflows, audit continuity and no cross-tenant data exposure.
- Reconcile or safely replay events/jobs/outbox according to idempotency rules. Do not replay external side effects blindly after restore.
- Record actual recovery start/end, restored point, observed RPO/RTO, data gaps, affected tenant scope, decisions and remediation actions.

## Restore drills and disaster scenarios

Run restore drills at a documented cadence and after material database/backup/topology changes. A successful backup is not recovery evidence until a restore has been exercised.

Test relevant scenarios:

- single tenant database corruption/deletion and isolated restore;
- full MySQL service/region/account loss according to architecture;
- accidental data change requiring approved point-in-time recovery;
- failed migration/backfill and compatible forward fix/rollback;
- Redis loss (rebuild cache/session helpers from MySQL safely);
- object storage/file metadata restore and authorization check;
- application redeploy/reconnect after recovery, including Hikari tenant registry, jobs/outbox and observability.

Drill results include target RPO/RTO, measured result, restoration evidence, defects, owner and next date. Any missed RPO/RTO is a tracked operational risk with remediation plan.

## Retention, archive and purge

- Every data category declares operational retention, legal/contractual retention, archive format/location, deletion method, owner and access controls: business data, audit logs, backups, application logs/traces, files, jobs/outbox and telemetry.
- Archive before purge only when mandated by policy/business. Archived data remains encrypted, tenant-scoped and restorable/readable for required period.
- Purge jobs are idempotent, bounded, observable, permissioned and tested on realistic volume. They do not cause table lock/outage or delete active/legal-hold records.
- Deletion/retention changes have audit evidence. A user/client data-deletion request is fulfilled through the product/legal policy—not by unsafe direct database deletion.

## Required alerts and evidence

Alert on missed/failed/late/suspiciously small backup, binary-log gap, storage/retention capacity, backup encryption/access failure, failed migration, tenant version drift and restore drill failure. Retain redacted backup/restore/migration reports, runbook execution, RPO/RTO results and approved exceptions with the operational release record.
