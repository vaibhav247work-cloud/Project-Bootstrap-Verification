# Cross-Stack CI, Release Evidence and Defect Management Standard

## One reproducible release candidate

A release candidate is a uniquely identified, immutable artifact built once from a reviewed commit and promoted through environments. Do not rebuild different binaries for SIT, UAT and production from the same source label.

Each artifact/release record identifies source revision, build ID/time, build environment, dependency lockfiles, generated API contract, database migration version, frontend bundle/version, configuration schema/version and container/package checksum where applicable.

## CI pipeline stages

```text
source/PR validation
  -> build/package from lockfiles
  -> fast static + unit/component checks
  -> API/schema/migration and MySQL/Redis integration checks
  -> security/supply-chain scans
  -> deployed contract/E2E/tenant-isolation checks
  -> performance/PWA checks when affected
  -> release candidate evidence bundle
  -> SIT/UAT approval and controlled production promotion
```

Run independent steps in parallel where practical, but never skip a dependency order: migrations before tests that need them; generated contract before client type check; packaged artifact before deployed system checks.

## Required gates

| Gate | Required evidence | Blocks release when |
| --- | --- | --- |
| Source/build | review, lockfile build, type/compile/lint/format | build is non-reproducible or required quality checks fail |
| Database/integration | migrations, MySQL/Redis tests, transaction/outbox/job tests | migration/integrity/cache/session/tenant failure |
| API/contract | OpenAPI diff and provider/consumer tests | unsupported breaking change or contract failure |
| Security/privacy | SAST, secrets, dependency/SBOM, scoped DAST/manual test | critical/high unresolved exploitable issue or secret/PII exposure |
| E2E/UAT | critical journeys, tenant isolation, PRD acceptance/UAT result | critical workflow, authorization or UAT blocker fails |
| Performance/recovery | required SLO/load/restore evidence | critical SLO, data safety or RPO/RTO gate fails |
| Release operations | deploy/rollback plan, monitoring/alerts, owner/on-call | no safe rollout, observability or recovery path |

The exact gate schedule is risk-based: fast checks on every PR, deeper checks on main/release candidate, and periodic scans/drills. A gate may be marked not applicable only with clear project reason, not merely to accelerate a release.

## Evidence bundle and retention

For each production release retain links or immutable artifacts for:

- source revision, approval/review record, build/provenance and deployed artifact checksum;
- CI results, test reports/coverage, contract diff, migration version and deployment configuration reference;
- redacted security/dependency/DAST reports, performance/load result and backup/restore evidence if required;
- E2E/UAT execution and sign-off, known defects, feature-flag state and approved exceptions;
- rollout start/end, monitoring result, rollback/recovery decision and release owner.

Set artifact retention according to client/compliance policy. Test evidence must be redacted/minimized: never retain raw production secrets, tokens, payment data or unnecessary PII in screenshots, logs, traces or test databases.

## Exceptions and risk acceptance

No individual developer silently overrides a failing gate. An exception records:

```text
gate and exact failure/risk
affected users/tenants/data and exploit/operational impact
why release is necessary now and compensating controls
authorized product/security/operations approver
owner, remediation ticket and expiry date
```

Expired exception automatically reopens the gate. Tenant isolation, authentication bypass, data corruption/loss, exposed secret and critical known exploitable security issues require the highest authorized escalation and normally block release.

## Defect lifecycle

Every defect uses a single tracked record with severity, priority, environment/version, affected tenant/user scope, reproduction, expected/actual outcome, evidence, owner and resolution verification.

| Severity | Example | Release rule |
| --- | --- | --- |
| Critical | tenant breach, auth bypass, data loss/corruption, outage of critical workflow | block/rollback immediately |
| High | major workflow unavailable/wrong result, exploitable serious vulnerability | block unless explicit exceptional risk acceptance |
| Medium | material workaround exists; limited business scope | schedule before/after release by agreed risk |
| Low | minor defect with low user/business impact | backlog with owner/priority |

- Fix verification reproduces original failure, validates the fix and runs appropriate regression suite.
- A defect found after release triggers incident process as needed, affected-scope assessment, user/client communication through authorized channels, root-cause review and prevention action.
- Any defect that automation could have caught adds/strengthens a regression test at the lowest effective layer.
- Track flaky tests as defects; retry metrics and quarantines are visible in release evidence.

## Promotion, rollback and post-release verification

- Promotion uses the same artifact and controlled configuration changes. Feature flags allow staged exposure but never replace tests/authorization.
- Define rollout scope, canary/staged criteria where appropriate, monitoring window, stop/rollback triggers, responsible owner and communications before production change.
- Rollback plan covers code, configuration, flags and database compatibility. Destructive/non-reversible migrations require forward-fix/restore plan before release.
- Post-release smoke checks validate health/readiness, version, core read/write path where safely authorized, authentication/tenant routing, dashboards/alerts and error budget/SLO signals.
- Review results after the monitoring window. Link incidents, regressions and follow-up tests/ADRs to the release record.

## CI platform security

- Use isolated, least-privilege CI identities and short-lived scoped credentials. Protected deployment secrets are unavailable to untrusted pull-request code.
- Pin/review build actions, base images and dependencies; protect branch/release permissions; log auditable deployment approvals.
- Aim for hermetic/reproducible builds from lockfiles. Generate and retain provenance appropriate to client risk; provenance makes an artifact traceable to the source and process that built it.
