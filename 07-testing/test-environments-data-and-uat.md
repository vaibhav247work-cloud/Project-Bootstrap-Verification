# Test Environments, Test Data and UAT Standard

## Environment purpose and separation

Each environment has one clear purpose, owner, access policy, data policy and reset/retention model. Environments do not share tenant databases, Redis namespaces, buckets, credentials or secrets with production.

| Environment | Purpose | Data rule |
| --- | --- | --- |
| Local | fast developer feedback | synthetic/disposable only |
| CI ephemeral | isolated automated unit/integration/contract tests | generated per run; destroyed after retention window |
| Shared dev/integration | team integration and provider sandbox checks | controlled synthetic seed; resettable |
| SIT/staging | production-like technical/system/release verification | anonymized or realistic synthetic, access restricted |
| UAT | business/client acceptance of approved release candidate | minimal controlled representative data, client-access restricted |
| Production | real service | never normal test target/data source |

SIT/staging should resemble production in architecture, security controls, observability, deployment packaging, tenant routing, schema/version and dependency behaviour in proportion to risk. It need not copy production scale/cost exactly; any meaningful difference is documented in the test report.

## Environment configuration and access

- Configuration is version-controlled by reference, environment-specific values externally supplied, and validated at startup/deployment. No production secret is copied into lower environments.
- Use unique environment identifiers in domains, emails, tenant codes, object-store paths, Redis key prefixes, audit/log indexes and outbound webhook headers so test traffic cannot be mistaken for production.
- Outbound email/SMS/payment/webhook/SSO integrations use sandbox, emulator or approved sink. A lower environment cannot send real customer communication, charge real money or modify client production systems by default.
- Access follows least privilege and is reviewed. UAT client users receive only required roles and only UAT tenant data.
- Environment health, version, migration state, active feature flags and known external dependency limitations are visible before a test run.
- Reset/reseed operations are documented, authorized and safe. Never reset a shared environment without verifying target/environment identity and notifying affected users.

## Test-data classification

Preferred order:

1. Fully synthetic generated data.
2. Anonymized/minimized production-derived data only when synthetic data cannot represent a required scenario.
3. Production data only for an explicitly approved, legally/privacy-reviewed and tightly controlled purpose—never copied casually.

Anonymization must be irreversible for the intended threat model; simple masking of visible fields is not sufficient if identities can be re-linked from combinations, IDs, free text, attachments or logs. Remove credentials, tokens, payment data, secrets, documents, biometric/health data and unnecessary sensitive fields. Apply access control, encryption, short retention, inventory and deletion confirmation.

## Seed-data rules

Maintain version-controlled factories/fixtures for realistic, minimal scenarios:

- at least two clearly separate tenants with colliding record IDs/names;
- active/disabled users, roles, custom permissions, temporary overrides and denied cases;
- lifecycle/status variants, concurrency/conflict data, time zones/locales and boundary dates;
- small/large dataset profiles, long text/files and empty/error states; and
- external-provider sandbox fixtures for success/failure/timeout/duplicate/schema change.

Data belongs to a named test run/tenant where possible. Test cleanup removes only owned data. Never include passwords, raw tokens, real email/mobile addresses or real personal data in fixtures, screenshots, reports or source control.

## UAT process

UAT confirms that the agreed product requirement works for real business users; it is not a substitute for developer integration, security or performance testing.

Before UAT starts, the release candidate has passed required CI, migrations, smoke checks, security/tenant gates and known-defect review. Product owner/client and team agree:

```text
UAT scope and acceptance criteria
test users/roles/tenants and data setup
environment URL/version/feature-flag state
test schedule, support contact and defect severity process
out-of-scope/known limitations and exit criteria
```

UAT scenarios use business language and map to PRD acceptance criteria. Each records precondition, actor/role/tenant, steps, expected outcome, actual result, evidence, tester and timestamp. Include happy, permission-denied, validation/error and important exception workflows.

## Defect and sign-off handling

- Log each UAT defect with reproducible steps, environment/version, tenant/test data, severity, owner, expected/actual result and evidence. Redact sensitive data.
- Severity is based on user/business/security impact, not just visual annoyance. Any tenant/security/data-integrity issue blocks release.
- A fix is retested in the same scenario and regression-tested at the lowest effective automated layer before UAT closure.
- Sign-off means agreed in-scope acceptance criteria passed and known residual risks/defects are explicitly accepted by the authorized client/product owner. It does not waive undiscovered security/legal/operational obligations.
- Retain UAT report, signed decision or equivalent audit trail with release evidence; apply the project retention policy.

## Production smoke tests

Production validation is separate from UAT. Default production smoke checks are health/readiness, version/configuration, dashboard/alert, safe read-only or dedicated reversible test-tenant flow. Any write/action in production needs explicit operational authorization, rollback/reversal and monitoring plan.
