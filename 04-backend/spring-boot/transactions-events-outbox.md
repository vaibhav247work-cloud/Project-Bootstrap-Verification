# Transactions, Events, and Outbox Standard

1. **MUST** place `@Transactional` at the application use-case boundary when multiple database writes form one business outcome.
2. **MUST** keep transactions short and never call email, SMS, payment, file storage, or third-party HTTP APIs inside a transaction.
3. **MUST** define rollback behaviour for checked exceptions and test failure paths.
4. **MUST** use database constraints as final protection for uniqueness/consistency; transactions do not replace constraints.
5. **MUST** write an audit event in the same transaction when accountability is required.

## Outbox

For a committed change that must trigger an external effect, write an `outbox_events` record in the same DB transaction:

```text
business update + audit event + outbox event → one transaction
worker reads outbox → publishes email/SMS/webhook/event → records outcome
```

Outbox records need event ID, tenant, type, payload/version, occurred time, attempt count, status, next attempt, and failure reason. Workers are idempotent; consumers must tolerate duplicate delivery. Never mark an event sent until the external result is known.
