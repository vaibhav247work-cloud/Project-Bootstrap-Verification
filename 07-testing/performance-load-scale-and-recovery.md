# Performance, Load, Scale and Recovery Testing Standard

## Capacity is a product decision

"Supports 1 million users" is not a testable requirement by itself. Before performance work, define measurable workload and service objectives:

```text
registered users and active users
peak concurrent users and requests/transactions per second
critical read/write workflows and expected mix
data volume per tenant/table/file/audit log and growth rate
p50/p95/p99 latency, error-rate and availability target
deployment topology, database/Redis/third-party limits
recovery point objective (RPO) and recovery time objective (RTO)
```

Targets are chosen from actual client/product risk. Do not copy internet benchmark numbers or promise capacity without a tested workload and infrastructure profile.

## Workload design

Load scripts are version-controlled code. They use production-like, anonymized/synthetic data and model real pacing/think time, authentication, role distribution, tenant distribution and request mix.

Every plan defines:

- endpoint/journey, expected volume/concurrency/duration and pass/fail SLO;
- warm-up, steady state, ramp up/down, test data setup/reset and safe stop condition;
- environment version/configuration/instance sizes and dependency simulation;
- dashboards/metrics/traces/database plans to capture; and
- owner, report location and remediation decision.

Test at least normal expected peak and a sensible headroom level. Never run destructive load against production unless the client has explicitly authorized scope, timing, limits and rollback/on-call plan.

## Test types

| Test | Goal | When required |
| --- | --- | --- |
| Baseline | establish current response/resource profile | first release and significant route/service change |
| Load | prove expected peak workload meets SLO | critical product workflow/release |
| Stress | find safe degradation/break point | capacity planning/high-impact system |
| Spike | validate sudden traffic burst/rate limiting/autoscale | campaigns, school result day, booking release etc. |
| Soak/endurance | find leak, pool exhaustion, queue/cache/connection drift | long-running or high-volume system |
| Scalability | compare capacity before/after added resources/data | scale claim or infrastructure change |
| Recovery | prove restore/restart/rollback and RPO/RTO | every production system at planned cadence |

## Application and API criteria

- Measure p50, p95 and p99 latency, throughput, error/status distribution, timeout/retry counts, queue time, saturation and user-visible completion—not only average latency.
- Separate expected business rejection (for example a valid 409 booking conflict) from platform failures, but measure both.
- Verify rate limits, overload shedding, bounded queues, timeout/retry/backoff, circuit behaviour and graceful error response under saturation.
- Critical idempotent commands are tested with retry/disconnect/duplicate load; verify one business result, correct audit/outbox and no hidden duplicate.
- Compare baseline to candidate; investigate meaningful regression even if an absolute budget has not yet failed.
- Frontend key routes additionally meet the documented field/lab Web Vitals target: default LCP <= 2.5 s, INP <= 200 ms and CLS <= 0.1 at p75, segmented by device class where data permits.

## MySQL, Redis and tenant fairness

- Run database load with representative schema, indexes, migrations and data distribution. Review slow queries, execution plans, lock waits/deadlocks, connection usage, CPU/memory/disk I/O, buffer/cache behaviour and replication/backup effects where applicable.
- Test pagination, search/filter/sort, reporting/export, JSON query paths and high-contention writes at realistic table size—not only empty/small fixtures.
- Exercise Hikari lazy tenant pool creation, steady use, eviction/recreation and pool exhaustion. A busy tenant must not exhaust global resources or cause cross-tenant data/context errors.
- Test Redis memory/eviction/TTL/key growth, cache hit/miss/stampede behaviour and outage fallback under load. Cache loss may reduce performance, never authorization correctness.
- Performance data has multiple tenant sizes and usage patterns: many small tenants, at least one large tenant, and concurrent tenants. Report per-tenant and total latency/error fairness where relevant.
- A scale result is tied to exact infrastructure, database version/configuration and dataset; it is not a permanent blanket claim.

## Data volume and reporting safety

- Use generated/anonymized records with realistic cardinality, skew, text length, dates and relationship density. Avoid all-identical fixtures that produce misleading query plans.
- Large exports/reports run with quota/limit/asynchronous job strategy where required; test that they do not block core transactions or exhaust memory.
- Audit logs, job/outbox and cache tables grow in tests as they grow in production; retention/archive/purge jobs are included in long-run validation.

## Recovery and operational drills

At a planned cadence, prove—not assume:

1. A backup can be restored into a clean isolated environment.
2. Restored schema/data/version/tenant boundaries and key business invariants are correct.
3. Measured restore time meets RTO and recoverable data point meets RPO.
4. Application startup/migration/health checks connect safely after restore.
5. Service/database/Redis process restart, failed deployment rollback and dependency outage recover without data loss or cross-tenant leakage.
6. In-flight jobs/outbox/retry/session/cache behaviour after recovery follows documented idempotency and expiry rules.

Record drill timestamp, source/target version, result, measured RPO/RTO, defects and remediation. A backup without a successful restore drill is not accepted as recovery evidence.

## Release gate and evidence

For a scale-sensitive release, retain workload scripts, seed approach, build/configuration identifiers, environment capacity, dashboard snapshots, bottleneck analysis, SLO result, known limits and remediation/exception approval. A release fails this gate when a critical workflow violates agreed SLO, corrupts/duplicates data, crosses tenant boundary, cannot degrade safely, or has no viable recovery evidence.
