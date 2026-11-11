# Redis Standard: Cache, Auth Support, and Tenant Isolation

## Role of Redis

Redis is an optional shared in-memory service for speed and short-lived coordination. MySQL remains the source of truth for business data, users, roles, audit records, refresh-session history, and durable revocation state.

If Redis is unavailable, the application must fail safely: protected requests may fall back to MySQL or be temporarily unavailable; it must never grant access because cache/session validation failed.

## Approved uses

| Use | Redis responsibility | Durable source of truth |
|---|---|---|
| Rate limiting | Counters/windows for login, reset, public API abuse | Not required; expiry is intentional |
| Permission/user cache | Short-lived effective-access or safe user summary cache | MySQL identity/RBAC tables |
| Session/revocation acceleration | Short-lived session-valid/revoked lookup cache | MySQL `user_sessions` state |
| Token/JTI denylist | Until the access JWT expires | Durable session revocation where needed |
| Distributed coordination | Short-lived locks/idempotency assistance | MySQL constraints/transactions remain final authority |
| Caching reads | Cached response/read model with TTL/invalidation | MySQL/domain source of truth |

## Prohibited uses

1. **MUST NOT** store plaintext passwords, raw refresh tokens, API secrets, private keys, or raw access JWTs in Redis.
2. **MUST NOT** make Redis the only durable record for financial, booking, identity, authorization, audit, or business data.
3. **MUST NOT** let a cache miss/error mean "authorized" or bypass tenant, role, or data-scope validation.
4. **MUST NOT** expose Redis directly to browsers, mobile apps, or the public internet.

## Authentication/session design

1. MySQL stores each refresh session and the refresh-token hash; it decides valid/revoked/expired status.
2. Redis **MAY** cache a minimal session-validation result or revoked session/token ID until its natural expiry.
3. On logout, password reset, role/security revocation, or account disable, update/revoke MySQL first; then evict/publish Redis invalidation.
4. A cached effective-permission result must have a short TTL and be evicted when user roles, permissions, overrides, organization membership, or account status changes.
5. Cache only the smallest safe data set: user ID, session ID, organization context, status/version, permission version/hash, and expiry. Do not cache sensitive profile data unless a specific performance need and privacy rule allow it.

## Key design and tenant isolation

Redis does not provide application namespaces; use a deliberate key convention. Include the `tenant_id` for every tenant-specific key when multiple tenants share the Redis deployment. Add `organization_id` only when the cached value is organization-specific beyond tenant scope.

```text
<optional-environment>:<app>:<purpose>:<optional-scope>:<entity>:<id>

erp:auth-session:session:01J...
erp:effective-access:tenant:xyzschool:user:987
erp:organization-settings:tenant:xyzschool:org:abc
erp:rate-limit:login:email-hash:ab12...
```

1. **MUST** derive tenant/organization scope IDs server-side, not from a client-provided key fragment, when scope isolation applies.
2. **MUST** prefix every key with an application identifier when Redis is shared by applications/services. Environment prefix is required only when environments share one Redis deployment; separate dev/staging/production Redis instances do not need it.
3. **MUST** avoid PII/raw email/token values in keys; hash them where correlation is required.
4. **MUST** set a TTL on every cache, rate-limit, lock, session-helper, and idempotency key unless a documented exception exists.
5. **MUST NOT** use Redis database-number selection as the primary tenant-isolation/security boundary.

## Security and operations

1. **MUST** restrict Redis network access to trusted application/worker hosts or private networking only.
2. **MUST** use Redis ACL users with least-privilege commands/key patterns; application workers must not have broad administrative commands.
3. **MUST** use TLS for Redis connections outside the same protected host/network boundary; use managed-service encryption at rest or platform/disk encryption according to the threat model.
4. **MUST** keep Redis credentials in a secrets manager/environment configuration, never source code.
5. **MUST** monitor memory, eviction, hit/miss rate, connection errors, latency, replication/availability, and key-expiry behaviour.
6. **MUST** define eviction policy and memory limits so Redis cannot exhaust the host and bring down the application.
7. **MUST** test cache invalidation, Redis outage, stale permission/session prevention, and rate-limit behaviour.

## When Redis is required

Redis is **not** required for every new product. Add it when measured/requested needs include distributed rate limiting, multiple application instances with shared short-lived state, high-read caching, background-job coordination, or a defined low-latency/revocation need. Record the decision in an ADR.
