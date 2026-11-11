# Container, Runtime, Health and Application Process Standard

## Deployment unit

Package the Spring Boot API/worker and React static frontend as immutable, versioned deployment artifacts. Containers/processes are stateless and disposable: durable business data lives in MySQL/object storage; short-lived shared state is governed by Redis policy.

Do not patch a running container or server manually. Rebuild from source, promote the artifact and deploy through the approved pipeline.

## Container image rules

- Use a multi-stage build and a minimal, trusted, pinned base image. Build tools, package managers, source repository and test files do not belong in final runtime image.
- Keep a strict `.dockerignore`; never copy `.env`, secrets, local credentials, build caches, test data or unrelated files into an image layer.
- Run as a dedicated non-root user. Use read-only filesystem and minimal Linux capabilities where the platform/application permits; writable temporary directories are explicit and bounded.
- Use explicit immutable image tag/digest for promotion. `latest` is never a production deployment reference.
- Image labels/metadata include product, module, source revision, build version/time and owner. Scan image/base/dependencies before promotion.
- One container has one primary process. Sidecars/init jobs are deliberate platform decisions, not a workaround for process management.
- Build and smoke-test the same image that will be deployed. Do not compile application code at container startup.

## JVM and Spring Boot process

- Set JVM memory/CPU behaviour, heap/metaspace/direct-memory limits, GC logging/diagnostics and timezone/encoding deliberately for the container resource limit. Do not rely on host defaults.
- Application logs are structured JSON to stdout/stderr with timestamp, level, service/version/environment, trace ID and safe tenant/user context. Do not write unbounded local log files in ephemeral containers.
- A process has one configuration/secret resolution path, validates it at startup and exposes build/version information safely through protected info/telemetry.
- Dev-only features, sample seed, debug endpoints, stack traces, broad CORS and permissive security are disabled in production profile.
- Workers/background schedulers are independently deployable/scalable when workload or operational risk requires it. If API and worker share a runtime initially, job concurrency/locks and shutdown behaviour are still explicit.

## Health endpoints and probes

Expose a narrow health contract for the runtime/orchestrator. Public internet access to management endpoints is prohibited; use private network, management port/path, firewall and least-privilege controls.

| Check | Meaning | Must not do |
| --- | --- | --- |
| Startup | application has completed initialization | repeatedly restart a slow but healthy app before its allowed startup window |
| Liveness | process is irrecoverably unhealthy and should restart | fail merely because a transient dependency/cache is unavailable |
| Readiness | safe to receive new traffic | report ready before mandatory config/security/runtime initialization is complete |
| Dependency indicators | current DB/Redis/provider condition for ops | expose secrets, topology, tenant names or sensitive diagnostics publicly |

- For Spring Boot, health groups/probes are explicit and tested. `liveness` normally proves the process/event loop; `readiness` proves this replica can serve its defined traffic.
- A single unavailable tenant datasource must not normally mark every application replica globally dead. Tenant-specific failure is surfaced through metrics/logs and handled per-request; platform-wide critical dependencies influence readiness only according to documented service policy.
- Never include all tenant database connections in a frequent health check; it causes connection storm and defeats lazy tenant datasource behaviour.
- Only approved Actuator endpoints are exposed; default to explicit allow-list. Never expose `env`, heap/thread dump, mappings, beans, config properties, logfile or mutable operations publicly.

## Resource management and scaling

- Every workload declares CPU/memory request/reservation and limit based on load evidence, JVM configuration and peak concurrency—not copied arbitrary values.
- Alert before saturation: CPU throttling, heap/GC pause, thread pool/queue, HTTP connection, Hikari pool, tenant datasource count, Redis/MySQL connections, file descriptors and disk/temp usage.
- Scale API replicas horizontally only when requests are stateless and tenant/session/cache/outbox rules support it. Do not assume adding pods solves a database/query/lock bottleneck.
- Autoscaling uses measured demand signals and safety bounds. Test scale-out, scale-in and cold-start under expected traffic; avoid rapid oscillation.
- Tenant datasource registry/pools are lazy, bounded and evicted only when idle/no borrowed connection as already defined in backend standards. Expose safe aggregate pool metrics—not raw tenant secrets—to operations.

## Graceful startup and shutdown

```text
startup: resolve config/secrets -> initialize app -> pass startup/readiness -> receive traffic

shutdown signal: mark unready -> stop new traffic -> finish bounded in-flight work
-> stop job intake -> close pools/clients -> terminate before platform grace period
```

- Orchestrator termination grace period exceeds Spring's shutdown timeout plus proxy/load-balancer drain allowance. These values are documented and tested.
- On shutdown, stop taking new HTTP work and new jobs, allow bounded in-flight idempotent work to complete, persist/requeue durable work safely, then close datasource/Redis/HTTP clients.
- Do not abruptly kill an application to deploy except during an approved emergency. Long-running work must be cancelable/checkpointed or run through durable background job design.
- Startup/shutdown is tested for active request, queued job, database connection, Redis outage and tenant pool eviction conditions.

## Runtime verification checklist

- Image digest/version and non-root/read-only capability checks pass.
- Container starts from clean environment with only declared config/secret references.
- Startup/liveness/readiness semantics are exercised, including dependency and slow-start cases.
- No sensitive management endpoint is externally reachable.
- Resource limit, JVM memory, graceful termination and autoscaling assumptions are load-tested.
- Logs/traces/metrics include version/correlation without tokens, secrets or unnecessary PII.
