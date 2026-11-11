# Notifications Module Standard

## Purpose

The Notifications Module provides a unified, multi-channel communication engine for sending transactional and lifecycle messages (Email, SMS, Push, In-App). It abstracts downstream providers, ensures tenant-scoped template isolation, respects recipient opt-out preferences, and maintains delivery accountability.

---

## 1. Multi-Channel Architecture

The module decouples business use cases from specific third-party notification APIs (e.g., SendGrid, AWS SES, Twilio, Firebase Cloud Messaging).

```text
Domain Event / Use Case
         ↓
Notification Dispatcher (determines channels & checks preferences)
         ↓
Tenant Template Engine (resolves localized message with Mustache/Thymeleaf)
         ↓
Transactional Outbox (`notification_outbox`)
         ↓
Asynchronous Notification Worker (rate-limited, pooled HTTP client)
   ├─ Email Provider Adapter (SES / SendGrid)
   ├─ SMS Provider Adapter (Twilio / MessageBird)
   ├─ Push Adapter (FCM / APNs)
   └─ In-App Channel (persists to `in_app_notifications`)
```

---

## 2. Core Operational Rules

1. **MUST** decouple notification delivery from user request threads:
   - Notifications must never block an active HTTP request or database transaction.
   - Publish to an outbox/queue and process via background workers with retries.
2. **MUST** enforce recipient preferences and opt-out rules:
   - Check `user_notification_preferences` before dispatching any marketing or optional notification.
   - Critical transactional notifications (password resets, MFA codes, security alerts, payment receipts) bypass marketing opt-out but must still be auditable.
3. **MUST** isolate templates by tenant:
   - In multi-tenant environments, custom email branding (logo, sender name, footer, localized text) is strictly scoped to the active `tenant_id`.
   - Dynamic templates must be sanitized and rendered using safe template engines (e.g., Mustache, Thymeleaf) that prohibit arbitrary code execution.
4. **MUST** implement channel-specific rate limits and throttling:
   - Protect downstream provider quotas and prevent email flooding during batch operations.
   - Throttling rules apply per recipient (e.g., maximum 5 password reset emails per hour per address).

---

## 3. Data Schema & Delivery Audit

Every notification dispatch creates an immutable delivery record:

```sql
CREATE TABLE notification_deliveries (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    tenant_id BIGINT UNSIGNED NOT NULL,
    user_id BIGINT UNSIGNED NULL,
    recipient_target VARCHAR(255) NOT NULL, -- normalized email or E.164 phone number
    channel VARCHAR(32) NOT NULL,           -- 'EMAIL', 'SMS', 'PUSH', 'IN_APP'
    template_code VARCHAR(64) NOT NULL,
    status VARCHAR(32) NOT NULL,            -- 'QUEUED', 'SENT', 'DELIVERED', 'FAILED', 'BOUNCED'
    provider_name VARCHAR(64) NOT NULL,     -- 'AWS_SES', 'TWILIO', 'FCM'
    provider_message_id VARCHAR(128) NULL,
    error_message VARCHAR(512) NULL,
    retry_count INT UNSIGNED NOT NULL DEFAULT 0,
    created_at DATETIME NOT NULL,
    delivered_at DATETIME NULL,
    INDEX idx_tenant_user (tenant_id, user_id),
    INDEX idx_status_created (status, created_at)
);
```

---

## 4. In-App Notifications

1. Store in-app notifications in a dedicated table `in_app_notifications` with `read_at DATETIME NULL`.
2. Deliver real-time updates via Server-Sent Events (SSE) or WebSocket only if the PRD requires instant UI badges.
3. Automatically expire or archive notifications older than 90 days.
