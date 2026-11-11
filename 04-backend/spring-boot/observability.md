# Observability Standard

1. **MUST** enable Spring Boot Actuator health, readiness/liveness, metrics and application info with production endpoint exposure restricted to intended operators.
2. **MUST** use structured logs containing timestamp, level, service/version, environment, trace ID and request ID; tenant/user identifiers are minimized/controlled.
3. **MUST** propagate trace/correlation IDs through HTTP clients, jobs and outbox events.
4. **MUST** record API latency/error rate, JVM, Hikari pool, MySQL, Redis, HTTP-client, worker/job and business critical-flow metrics.
5. **MUST** create alerts for health failure, error-rate increase, latency, pool exhaustion, job backlog/failure, backup failure and authentication abuse.
6. **MUST NOT** expose Actuator env/config/heap/thread/database details publicly.
7. **MUST** redact secrets, tokens and sensitive data in logs/traces/metrics; avoid unbounded-cardinality labels such as user IDs or raw request URLs.

Operational dashboards and alerts are tested during staging/release verification, not only after an incident.
