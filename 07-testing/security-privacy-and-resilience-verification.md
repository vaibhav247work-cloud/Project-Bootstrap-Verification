# Security, Privacy and Resilience Verification Standard

## Baseline

Use the current approved OWASP ASVS version as a security-verification reference; record the exact version and applicable level/scope in each product security plan. OWASP ASVS is a verification framework, not a checkbox replacement for product threat modelling or expert review.

Security tests are run only against owned, explicitly authorized non-production targets unless a separately approved production test plan exists. Never conduct invasive scanning against a client/third-party environment without written scope and timing approval.

## Mandatory security verification

| Area | Required negative tests |
| --- | --- |
| Authentication/session | invalid/expired/wrong issuer JWT, revoked session, refresh reuse, idle/absolute expiry, logout/reset/disable revocation, CSRF when cookies authenticate requests |
| Authorization | default deny, missing permission, wrong role/scope/owner/state, guessed ID/direct endpoint, static/file download and admin action enforcement |
| Tenant isolation | forged/missing context, cross-tenant IDs, cache/datasource/job/webhook/file boundary tests from the tenant isolation standard |
| Input/output | invalid types/bounds, injection/XSS/unsafe URL/rich content, mass-assignment/unknown field and malicious file cases |
| API/browser transport | CORS origin/credential behaviour, security headers/CSP, clickjacking, open redirect, rate limit/abuse response, safe errors |
| Secrets/crypto | no secret/token/password in source/build/artifact/log/telemetry, key/credential rotation failure path, approved password/token handling |
| Dependencies/supply chain | SCA/SBOM/license policy, lockfile/reproducible build, third-party script/package review |

Every security-sensitive endpoint has both positive and negative tests. A successful privileged test never proves that an unauthorized user is denied.

## Authorization and business-abuse tests

- Test permission, object/relationship scope, tenant context and lifecycle/state separately and together. For example, a teacher may view their own class but not another school's record even with a valid guessed ID.
- Test temporary direct override start/expiry/revocation, approver/reason/audit, and ensure it does not become a permanent implicit role.
- Test high-risk workflows for step skipping, parameter tampering, race conditions, duplicate submit, stale approval, price/quantity manipulation and replay.
- Test rate limit/lockout/captcha/abuse controls from a safe test profile. Verify appropriate 429/retry guidance, audit/alert and no account enumeration.
- Verify sensitive failure messages do not disclose account existence, tenant membership, role, hidden resource or internal implementation details.

## Privacy and log verification

- Create privacy-focused test fixtures containing fake tokens, passwords, PII and sensitive-looking values. Search all HTTP responses, logs, audit events, traces, metrics, analytics, browser storage, caches, backups/test artifacts and error reports for leakage.
- Expected rule: audit records capture actor/action/time/context safely; technical logs capture diagnostic metadata; neither stores raw passwords, access/refresh tokens, secrets, full sensitive request bodies or unrestricted personal data.
- Verify data minimisation in UI/API response, consent/preference rules where applicable, retention/expiry/deletion jobs and tenant/user export/delete workflows if the product offers them.
- Verify access to logs, audit history, backups and test artifacts is role-restricted and retention controlled.

## Automated scanning and review

- Run SAST, secret scan, dependency/SBOM vulnerability scan and infrastructure/container scan in CI appropriate to the delivered components.
- Run authenticated DAST/API scanning in a disposable/stable non-production environment on a scheduled/release cadence. It supplements, never replaces, targeted manual tests.
- Findings are triaged by exploitability, exposure, data/tenant impact and compensating controls. Critical/high release blockers are fixed or have explicit time-bound risk acceptance from authorized owner.
- False positives are documented with rationale and revalidated when code/dependency/environment changes.

## Resilience and failure verification

For every critical dependency and state transition, test normal operation plus controlled failure:

```text
MySQL/tenant datasource unavailable or slow
Redis unavailable, stale or evicted
third-party timeout/429/5xx/malformed response
duplicate/out-of-order event or job retry
process restart during command/outbox/job
network disconnect/reconnect and client retry
deployment/version mismatch or lazy-chunk/PWA update failure
```

- Verify timeouts, bounded retry/backoff, circuit/fallback, user-safe errors, trace/audit visibility and recovery—never infinite retry or silent data loss.
- Verify transactions/outbox preserve atomic business/audit/event state. Duplicate delivery/retry is safe through idempotency.
- Verify degraded components fail closed where security requires it. Redis/cache failure must not grant access; tenant routing failure must not fall back to another tenant.
- Perform controlled restart/rollback/recovery drills for critical services and jobs. Confirm in-flight command/draft/queue behaviour is defined and observable.
- Test backup restore and data-integrity checks under the database operations standard; a backup that has never been restored is not verified recovery.

## Evidence and release gate

Retain scope, ASVS reference/version, automated reports, manual scenario results, redacted findings, retest evidence and risk acceptances with the release record. Release is blocked by unresolved critical tenant-isolation, authentication, authorization, secret exposure, known exploitable high-risk vulnerability or untested critical recovery path unless the authorized business/security owner explicitly accepts a time-bound exception.
