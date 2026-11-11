# Audit Trail and Compliance Service Standard

## Purpose

The Audit Trail Service provides an immutable, tamper-evident log of security-sensitive events, administrative changes, and transactional mutations across all product modules. It serves compliance, forensics, and dispute resolution.

An audit log is distinct from technical application logs: **technical logs record system health and errors; audit logs record business and identity accountability.**

---

## 1. What Must Be Audited

1. **Authentication & Session Events:**
   - Login success/failure, logout, session revocation, MFA verification, password resets, account lockouts.
2. **Access Control & Permissions:**
   - Role assignments/removals, permission grant modifications, temporary override creation/expiration.
3. **Sensitive Business Mutations:**
   - Financial transactions, salary changes, medical/PII record modifications, status transitions on critical workflows (orders, bookings, approvals).
4. **Data Exports & Impersonation:**
   - Any bulk download/export of tenant records, support impersonation ("Login As") sessions.
5. **System Configuration:**
   - Feature flag toggles, integration key changes, tenant settings updates.

---

## 2. Core Immutability & Security Rules

1. **MUST** be append-only:
   - Database user/role powering the application must have `INSERT` and `SELECT` only on audit tables; `UPDATE` and `DELETE` grants are strictly prohibited in database permissions.
2. **MUST NOT** store credentials or raw secrets:
   - Never log passwords, raw access/refresh tokens, full credit card numbers, or encryption keys in audit payloads.
3. **MUST** record complete actor attribution:
   - `user_id` of the caller.
   - `actor_type`: `USER`, `SYSTEM_JOB`, `SERVICE_ACCOUNT`, `SUPPORT_AGENT`.
   - `impersonated_by_user_id`: Present if an administrator acted on behalf of a tenant user.
   - `source_ip` and `user_agent` (subject to privacy data minimization).
4. **MUST** commit in the same transaction as the business operation:
   - For critical database mutations, the audit log record must be inserted in the exact same database transaction as the entity update (`04-backend/spring-boot/transactions-events-outbox.md`). If the transaction rolls back, no false audit event is recorded.

---

## 3. Standard Audit Log Schema

```sql
CREATE TABLE audit_events (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    tenant_id BIGINT UNSIGNED NOT NULL,
    trace_id VARCHAR(64) NOT NULL,
    actor_id BIGINT UNSIGNED NOT NULL,
    actor_type VARCHAR(32) NOT NULL,              -- 'USER', 'SYSTEM_JOB', 'SUPPORT_IMPERSONATION'
    impersonated_by_id BIGINT UNSIGNED NULL,
    action VARCHAR(64) NOT NULL,                  -- 'employee.salary.update', 'user.role.assign'
    target_type VARCHAR(64) NOT NULL,             -- 'Employee', 'Role', 'Booking'
    target_id VARCHAR(128) NOT NULL,
    state_before JSON NULL,                       -- sanitized snapshot before mutation
    state_after JSON NULL,                        -- sanitized snapshot after mutation
    diff_summary VARCHAR(512) NULL,               -- human-readable delta
    source_ip VARCHAR(45) NULL,
    user_agent VARCHAR(255) NULL,
    created_at DATETIME NOT NULL,
    INDEX idx_tenant_action_time (tenant_id, action, created_at),
    INDEX idx_target (tenant_id, target_type, target_id),
    INDEX idx_actor (tenant_id, actor_id)
);
```

---

## 4. Retention and Compliance Exports

1. **Retention:**
   - Default retention period: 2 years (or contractual/regulatory compliance target, e.g., 7 years for financial records).
   - Old audit partitions moved to tamper-evident object storage (e.g., S3 Object Lock in Compliance Mode) before archiving.
2. **Audit Query & Export API:**
   - Expose tenant-scoped audit search APIs strictly protected by elevated permissions (`audit.events.view`, `audit.events.export`).
   - Tenant administrators can only query their own `tenant_id` records.
