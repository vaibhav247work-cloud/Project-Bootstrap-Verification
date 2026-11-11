# MySQL Audit Log Standard

## When audit is mandatory

Audit records are mandatory for authentication, role/permission/override changes, financial events, approvals, status changes, sensitive-data access or export, and administrative configuration changes. Projects may add more events.

## Reference record

```text
audit_logs
  id
  occurred_at
  organization_id
  actor_type                 -- user, system, integration
  actor_user_id              -- nullable only for a non-human actor
  event_type                 -- e.g. employee.updated
  entity_type
  entity_id
  action                     -- create, update, delete, export, approve, login, ...
  outcome                    -- success, failure
  request_id
  source_ip                  -- apply privacy/retention policy
  reason
  before_data                -- redacted/minimized JSON when needed
  after_data                 -- redacted/minimized JSON when needed
  metadata                   -- safe technical/business context
```

## Rules

1. **MUST** make audit records append-only for application users. No regular application API may update or delete an audit record.
2. **MUST** record actor, time, event/action, target, outcome, organization, and request/correlation ID where available.
3. **MUST** distinguish user, system job, and integration actors.
4. **MUST NOT** store passwords, access/refresh tokens, secrets, full payment credentials, or unnecessary sensitive data in audit fields.
5. **MUST** mask/minimize before/after data. Store a field-level summary when full values are not necessary for accountability.
6. **MUST** protect audit-log read access with a dedicated permission and record audit-log exports/access where the risk requires it.
7. **MUST** define retention, archive, and legal-hold requirements per project; backup copies are included in the retention design.
8. **SHOULD** write the business change and its audit event in the same transaction when losing the audit event would make the change unaccountable.

`created_by_user_id` and `updated_by_user_id` answer who last changed a row. They do not replace audit history.
