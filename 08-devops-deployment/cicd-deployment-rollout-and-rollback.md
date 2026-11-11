# CI/CD, Deployment, Rollout and Rollback Standard

## Build once, promote unchanged

CI builds one immutable release artifact from reviewed source and locked dependencies. The exact artifact digest/version is promoted through dev, SIT, UAT and production; only approved environment configuration and secret references differ.

```text
reviewed commit -> tested artifact + SBOM/provenance -> registry digest
-> deploy dev/SIT -> UAT approval -> controlled production promotion
```

Never rebuild a "production version" from the same commit manually, deploy a mutable `latest` tag, or change running code through SSH/console edits.

## CI/CD pipeline ownership

- Pipeline definitions, deployment manifests/IaC, migration process and environment configuration references are version-controlled, reviewed and protected like application code.
- CI identity can build/test/push only required registries; deployment identity can deploy only approved environment/namespace/account. Pull-request code never receives production secrets or broad deployment credentials.
- Artifact registry retains release/recent rollback images according to retention policy and denies unauthenticated public write/read unless explicitly required.
- Each artifact has source revision, build ID/time, SBOM, image/package digest and test/security evidence link. Sign/provenance is enabled where the platform/client risk warrants it.
- Pipeline concurrency/locks prevent two production deployments or incompatible migrations from racing. Emergency deployment remains audited and reconciled back to source of truth.

## Deployment preconditions

Before promoting an artifact:

1. Required gates from `07-testing/` pass for the exact artifact.
2. Environment configuration/secret references validate without exposing values.
3. Migration/API/client backward compatibility is reviewed.
4. Feature flags, rollout scope, monitoring dashboard, alert owner, support contact and rollback/forward-fix plan are ready.
5. Change is scheduled/communicated according to client risk and maintenance requirements.

## Database migration sequencing

Use expand/contract migration choreography:

```text
1. add backward-compatible schema/data path
2. deploy code compatible with old and new schema
3. backfill/verify asynchronously and safely
4. switch reads/writes after verification
5. remove old path only in a later reviewed release
```

- Migrations are versioned, idempotent as appropriate, logged and executed once by an approved migration job/process—not concurrently by every application replica.
- Before destructive/non-reversible schema change, take/verify backup, confirm restore path, record data-impact/lock-time, and have forward-fix or restore plan.
- Database-per-tenant migrations have inventory, tenant version/status, bounded concurrency, retry/recovery, observability and a clear policy for failed tenant migration. Never silently deploy application code that requires a schema absent from a tenant database.

## Rollout strategy

Default to rolling deployment with readiness gates. Use blue/green or canary/staged rollout when blast radius, traffic, schema risk, client impact or service criticality justifies it.

For each production rollout define:

- initial traffic/tenant scope, increment schedule and observation window;
- success metrics: readiness, error rate, p95/p99 latency, saturation, business success, login/permission/tenant error signals;
- stop/rollback triggers, authority and communication path;
- affected feature flags/configuration and browser/frontend asset/cache compatibility; and
- validation journey using safe production smoke test or approved dedicated test tenant.

Do not mark a deployment successful solely because pods/instances are running. It is successful when readiness, critical journeys and defined operational/business signals remain healthy through the observation window.

## Rollback and forward fix

- Rollback is a tested, one-command/controlled pipeline action to a known artifact/configuration version where possible.
- Code rollback is safe only when database/API/event/feature changes remain compatible. If not, use documented forward fix, flag disable or restore procedure.
- A rollback never deletes unknown data or replays irreversible actions blindly. Audit/outbox/job consumers remain idempotent across mixed-version period.
- Frontend deployment keeps prior versioned static assets available for a safe window so open browser tabs/service workers can load chunks; backend API supports the rolling client compatibility window.
- After rollback, verify health, tenant routing, security/session, core workflow, migration state and dashboards. Preserve diagnostics before cleanup.

## Deployment verification

Automated deployment checks confirm artifact digest, target environment, configuration schema, migration state, startup/readiness, security headers/Actuator exposure, static frontend version and smoke journey. Manual approval is required only at designated risk gates and records approver, reason and artifact.

Deployment logs include timestamp, actor/pipeline ID, artifact/config/migration version, environment, rollout actions and result; they never expose secret values.

## Required drill and evidence

- Perform a non-production full promotion and rollback rehearsal before first production release and after material pipeline/topology/migration changes.
- Periodically test production rollback/feature-disable/migration recovery in a safe approved manner.
- Retain CI/deployment/migration/rollout/rollback evidence with release record. Treat a failed deployment, manual hotfix or unexpected rollback as an improvement item with root cause and automation/runbook follow-up.
