# Observability, Alerting, SLO and Dashboard Standard

## Outcome

Operations must answer quickly: Is the service healthy for users? Which tenant/module/version is affected? What changed? Where is the bottleneck? Can we recover safely? Observability is designed with the product, not added after an incident.

Use correlated structured logs, metrics and traces. OpenTelemetry is the default interoperability model; the chosen backend/vendor may vary by project.

## Correlation and telemetry context

Every inbound request gets a trace/correlation ID. Propagate it through HTTP client calls, asynchronous job/outbox events and approved message metadata. Log/trace fields include safe service name, version, environment, module/operation, outcome and timing.

- Tenant/user/session identifiers are never raw PII/secrets in logs or metric labels. Use a safe internal code/hash only when necessary for controlled diagnostics.
- Do not put access/refresh tokens, passwords, cookies, authorization headers, secrets, raw payment/identity data, full request/response bodies or unbounded free-text in telemetry.
- Context propagation is explicit and cleared after request/job completion. A worker thread must not reuse user/tenant context from an earlier task.
- Frontend telemetry follows the browser security/privacy standard: consent/data minimisation, redaction, sampled errors/vitals, build version and safe route category—not sensitive URL/query/body data.

## Structured logs

Application logs are structured, timestamped and emitted to the platform collector. Minimum fields:

```text
timestamp, severity, service, module, environment, version/build_id,
trace_id, operation/route, outcome/status, duration, safe error code
```

- Log each unexpected exception once at global boundary with trace and safe diagnostic cause; do not duplicate stack traces at every layer.
- Security/business audit records remain distinct from technical logs and follow their own retention/access policy.
- Log levels are controlled configuration with a safe production default. Temporary debug logging is scoped, approved, time-limited and reviewed for sensitive data exposure.
- Set retention, access control, encryption, query audit and deletion policy for logs/traces. Alert when telemetry pipeline ingestion/delivery itself fails.

## Metrics

Use unit-bearing, consistently named counters, gauges and histograms. Standard service metrics include:

| Area | Minimum signals |
| --- | --- |
| HTTP/API | request count, status/outcome, latency histogram, active requests, request/response size where useful |
| JVM/runtime | CPU, memory/heap/GC, threads, file descriptors, process restarts, container throttling/limits |
| Hikari/MySQL | acquisition/wait, active/idle/pending connections, query/transaction errors/latency, pool/datasource eviction/failure |
| Redis/cache | connection/latency/error, hit/miss, evictions, memory, rate-limit/session/cache invalidation result |
| HTTP providers | calls, latency, status/error, retry/circuit state, timeout/rate-limit result |
| Jobs/outbox | queue/in-progress/retry/dead-letter, age/lag, last success/failure, duration, processed count |
| Business | successful/failed core command count, conflict/abandon signals, never unnecessary personal data |

Metric labels remain low-cardinality and bounded: status class/code, route template, operation, module, environment, version. Never label by user ID, email, session/token, raw tenant ID where tenant count is unbounded, request ID, booking/order ID, URL query or exception message. For tenant-specific diagnostics, use logs/traces or a controlled top-N/aggregate dashboard rather than unbounded metric time series.

## Traces

- Trace critical inbound routes, database/Redis/external calls, authentication/authorization decision boundary, migrations, jobs and outbox delivery.
- Span names are stable low-cardinality operations, not full URLs/SQL/personal data. Add safe attributes such as module, outcome and retry count.
- Sampling is deliberate: retain errors and slow/high-value traces at higher rate, sample normal high-volume traffic within cost/privacy budget. Tail-sampling policy is documented.
- Traces link safely to logs/metrics through trace ID and exemplars where platform supports it.

## SLI, SLO and error budget

Each client-facing critical service/journey declares:

```text
SLI: measured good events / total valid events
SLO: target and rolling period
error budget: tolerated bad events
scope: route/journey, tenant/class, environment and exclusions
owner: team/person and escalation path
```

Examples include successful authenticated API request latency, booking-confirmation success, login availability or job completion freshness. Choose objectives from business/user impact—not a universal number. Exclude only explicitly justified client-aborted/test/invalid cases; never hide real server failures.

Error-budget consumption informs release pace and remediation priority. An SLO without a credible measurement/query/dashboard is not accepted.

## Dashboards

Every production service has owned dashboards for:

1. User/service overview: traffic, good/bad events, latency percentiles, SLO/error-budget and current release version.
2. Dependency health: MySQL, Redis, external APIs, queue/jobs and datasource/Hikari state.
3. Runtime capacity: instance count/readiness/restarts, CPU/memory/GC, saturation and deployment rollout.
4. Security/access: authentication failures/rate-limit, authorization denial anomalies, session/revocation/cache errors—without exposing sensitive identities.
5. Business workflow: core success/failure/conflict/backlog signals where meaningful.

Dashboards are version-controlled or have documented owner/export/backup. A dashboard panel has clear unit, aggregation/window and link to relevant runbook.

## Alerting

Page only when a human must act now to protect users/data/SLO. Alerts must be actionable, deduplicated/grouped, routed to named owner/on-call, severity-labelled and include dashboard, trace/log query, runbook and rollback/escalation link.

- Prefer alerts based on significant SLO/error-budget burn or sustained user-impact symptoms over raw host noise.
- Add warning/ticket alerts for capacity trends, certificate/secret expiry, backup/restore failure, stale job, rising error/retry and security scan finding before they become an incident.
- Avoid alerts without owner, runbook or clear action. Tune false positives/false negatives after incidents; test alert delivery/routing periodically.
- Alert on data integrity/tenant boundary/authentication bypass/security exposure immediately according to incident policy even if general error rate is low.

## Required verification

- Automated tests verify correlation propagation, redaction, metric names/label cardinality and key alert rules.
- Every new critical endpoint/job/dependency has appropriate metrics/traces/log outcome before release.
- Deployment can be observed end-to-end by build/version and rollback verified through dashboard signals.
- Periodically run an alert/fire drill and confirm owner notification, dashboard/runbook usefulness and escalation timing.
