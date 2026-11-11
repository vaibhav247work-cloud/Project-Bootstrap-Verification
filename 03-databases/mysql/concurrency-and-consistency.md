# MySQL Concurrency and Consistency Rules

## Goal

Two simultaneous requests must not create duplicate bookings, double payments, lost edits, invalid inventory, or untraceable partial updates.

## Mandatory rules

1. **MUST** identify concurrency-sensitive workflows during feature design: booking/seat allocation, inventory, payment, approvals, counters, and simultaneous editing.
2. **MUST** enforce duplicate-prevention business rules with database `UNIQUE` constraints, not only a frontend check.
3. **MUST** use a database transaction when multiple data changes represent one business outcome.
4. **MUST** keep transactions short and avoid calls to email, payment, file, or external APIs inside them.
5. **MUST** define retry/idempotency behaviour for externally retried write requests, webhooks, imports, and payment callbacks.
6. **MUST** turn unexpected duplicate-key/deadlock/concurrency failures into a clear, safe API result; never silently retry an uncertain non-idempotent operation.

## Optimistic locking: default for normal editing

For frequently edited user-facing records, add a `version` column.

```text
Read:   employee id=42, version=7
Update: succeeds only if version is still 7; then sets version=8
```

If another user saved first, return a conflict response and let the user refresh/review. This prevents a quiet last-write-wins data loss.

## Pessimistic locking: exceptional use

Use a row lock only for a short, clearly defined critical section where a resource cannot be allocated twice (for example, the last available seat). Document lock order and timeout/retry behaviour. Do not use locks as a substitute for correct uniqueness constraints.

## Idempotency

For a client/API operation that may be repeated, use a caller-supplied idempotency key scoped to the user/organization and action. Store enough request/result state to return the original result rather than creating a second outcome.

```text
idempotency_keys
  organization_id, actor/user identity, operation, key, request_hash,
  status, response reference, expires_at, created_at
```

Add a uniqueness constraint appropriate to the scope, for example `(organization_id, operation, key)`.

## MySQL transaction policy

InnoDB is required. Its default isolation level is `REPEATABLE READ`; do not change the global level casually. Select a different per-transaction isolation level only with a documented consistency/performance reason and tests.
