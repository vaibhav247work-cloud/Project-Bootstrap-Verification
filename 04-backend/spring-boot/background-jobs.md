# Background Jobs and Scheduling Standard

Use durable jobs for export/import, notifications, outbox publishing, reports, file processing, archive cleanup and retryable external work. `@Async` is not a durable queue.

1. **MUST** persist durable jobs/events before processing; after restart, queued/in-progress work can be recovered safely.
2. **MUST** use bounded named executors/worker pools; never use unbounded threads or the common pool for production work.
3. **MUST** propagate validated tenant context, trace ID and actor/system identity explicitly into every job.
4. **MUST** make jobs idempotent, retry with bounded exponential backoff/jitter, and send exhausted failures to visible failed-job/dead-letter handling.
5. **MUST** prevent duplicate scheduler execution across application instances using a DB/Redis lease or a dedicated worker/scheduler deployment.
6. **MUST** record job type, tenant, status, timing, attempts, failure reason and safe result reference; never log secrets/full sensitive payloads.
7. **MUST** provide cancellation/expiry/retention rules for long-running exports/imports.

For simple local periodic work Spring `TaskScheduler` is allowed; distributed production schedules still need a single-execution lock/lease.
