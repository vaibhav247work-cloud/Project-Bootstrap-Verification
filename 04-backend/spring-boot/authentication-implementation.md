# Spring Boot Authentication Implementation Standard

## Scope

Implements the existing email/password, optional passkey/SSO, JWT access-token, rotating refresh-session, Redis, RBAC, and audit policies. Do not create ad-hoc authentication logic outside this module.

## Package boundary

```text
security/
  config/             SecurityFilterChain, CORS, CSRF, encoders, JWT configuration
  authentication/     login, refresh, logout, password/reset/verification flows
  session/            refresh-session persistence, Redis cache/invalidation
  authorization/      permission, scope, and ownership evaluators
  tenant/             current tenant context (only when product has tenant context)
  audit/              security/business audit-event writer
  web/                controllers, request/response DTOs, error handlers
```

Controllers handle HTTP only. Authentication/session rules live in services; persistence is behind repositories. Never return entities, password hashes, raw tokens, or internal security details from a controller.

## SecurityFilterChain policy

1. **MUST** use one explicit `SecurityFilterChain`; no accidental framework default-login screen/basic auth in production.
2. **MUST** allow only documented public routes: health/readiness as appropriate, login, refresh, logout, registration/invitation/reset/verification, and OIDC callback routes when enabled.
3. **MUST** require authentication for all remaining API routes, then enforce permission/scope in the application authorization layer.
4. **MUST** configure CORS from explicit environment configuration; never use wildcard origins with credentialed cookies.
5. **MUST** configure CSRF protection for cookie-authenticated refresh/logout or other cookie-authenticated endpoints; bearer-only API endpoints follow the documented token model.
6. **MUST** use JSON `AuthenticationEntryPoint` (401) and `AccessDeniedHandler` (403), without leaking account/token details.
7. **MUST** use a stateless security context for access-JWT API authentication; refresh-session state is server-side.

Spring Security's filter chain authenticates before authorization. Use its standard resource-server/JWT support where it fits, with custom validators/converters only for project-specific claims.

## Password encoder

1. **MUST** expose a production `PasswordEncoder` using Argon2id where the selected supported Spring Security/library version permits it; parameters are benchmarked on deployment hardware.
2. **MUST** retain an algorithm identifier/encoded format so existing hashes can be verified and upgraded on successful login.
3. **MUST NOT** use `NoOpPasswordEncoder`, demo passwords, or hard-coded credentials in production.

## Authentication endpoints

| Endpoint | Purpose | Notes |
|---|---|---|
| `POST /api/v1/auth/login` | Email/password or selected login start | Rate limited; generic public failure response |
| `POST /api/v1/auth/refresh` | Rotate refresh token, issue access JWT | Refresh cookie; CSRF protected for browser |
| `POST /api/v1/auth/logout` | Revoke current session | Clear refresh cookie and Redis helper key |
| `POST /api/v1/auth/logout-all` | Revoke all user sessions | Requires recent auth for sensitive products |
| `POST /api/v1/auth/forgot-password` | Start reset | Generic response; rate limited |
| `POST /api/v1/auth/reset-password` | Consume reset token | Revokes sessions on success |
| `POST /api/v1/auth/verify-email` | Consume verification token | Single-use token |
| `GET /api/v1/auth/me` | Current safe user/session/permission summary | Never source of backend authorization truth |

Passkey and SSO endpoints are enabled only when their PRD features are enabled.

## Login flow

```text
Request → validation/rate-limit → load active user → password/passkey/SSO proof verify
→ create MySQL user_session + hashed refresh token → Redis short-lived session helper
→ write audit event → access JWT response + Secure/HttpOnly refresh cookie
```

After login, validate `users.status`; suspended/disabled users never receive a valid session. A token may contain `sub`, `sid`, `jti`, `iat`, `exp`, `iss`, `aud`, `typ`, and an optional configured tenant context. It must not contain the complete permission list or sensitive profile data.

## Per-request access-JWT flow

```text
Bearer JWT
→ validate issuer/audience/signature/allow-listed algorithm/type/expiry
→ extract user ID + session ID
→ verify active user and non-revoked session using Redis helper
→ safe MySQL fallback if Redis cache misses
→ establish SecurityContext
→ controller/service performs permission + data-scope + business-state check
```

Do not query MySQL on every request only to rebuild permissions. Use a short-lived Redis effective-access cache, evicted whenever user status, roles, permissions, or override data changes. A cache problem must fail closed or use the MySQL authority; it cannot grant access.

## Refresh and logout

1. Refresh reads the opaque `HttpOnly`, `Secure`, appropriately `SameSite` cookie; it never accepts a raw refresh token in query parameters.
2. In one transaction, validate the stored hash/state/idle and absolute expiry, rotate the refresh token, update session last-use state, and emit audit data.
3. Refresh-token reuse/revocation triggers session-family revocation, Redis eviction, audit, and a 401 requiring login.
4. Logout updates MySQL session state first, then evicts Redis. Clearing the browser cookie alone is insufficient.

## Database-per-tenant routing

For database-per-tenant multitenancy, a validated tenant context selects the correct Hikari datasource at runtime, for example `tenant_xyzschool` or `tenant_jhkschool`.

```text
XYZ request: Spring Boot → tenant_xyzschool + Redis keys tenant:xyzschool:...
JHK request: Spring Boot → tenant_jhkschool + Redis keys tenant:jhkschool:...
```

Use the JPA/Hibernate implementation in `database-per-tenant-jpa.md`. A tenant selector/central identity layer is needed only when a future product needs one login across multiple tenant databases; record it as an ADR.

## API errors

Use a consistent Problem Details-style JSON response. Keep public authentication messages generic; record specific internal reason codes only in protected audit/technical logs.

```json
{
  "type": "https://api.example.com/problems/authentication-failed",
  "title": "Authentication failed",
  "status": 401,
  "traceId": "..."
}
```

See `global-error-handling.md` for mandatory exception mapping, safe logging, security errors, and trace IDs.

## Required implementation tests

- Valid/invalid/expired/wrong issuer or audience JWTs
- Revoked session and Redis cache-miss/fallback behaviour
- Refresh rotation, reuse detection, idle and absolute expiry
- Logout current/all, password reset, disabled/suspended account
- CORS/CSRF behaviour for cookie refresh/logout endpoints
- Password rehash upgrade path
- Tenant deployment key namespace and datasource configuration
- Authentication and authorization audit events contain no raw secrets/tokens
