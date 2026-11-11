# Token and Session Policy

## Default model

Use a short-lived JWT access token plus a server-tracked, rotating refresh token.

```text
Login / refresh
  -> short-lived JWT access token
  -> opaque refresh token tied to one server-side session

API request
  -> validates JWT signature, issuer, audience, expiry, subject, session ID, and active tenant context
  -> performs current authorization server-side
```

This model supports logout, device/session management, compromise response, and multi-device use without putting long-lived credentials in JavaScript-readable storage.

## Default durations

| Token/session | Default | Project may change when |
|---|---:|---|
| Access JWT | 15 minutes | Higher-risk workflows may use 5 minutes; a different value needs rationale |
| Refresh session idle timeout | 30 days | Client/user experience or risk profile requires another period |
| Refresh session absolute lifetime | 90 days | Client policy requires a shorter/longer controlled period |
| Password-reset / email-verification token | Short-lived, single use | Exact period selected by authentication feature policy |

All expiry and inactivity decisions are enforced by the server. For finance, admin/security configuration, or other high-risk actions, require recent authentication/re-authentication regardless of normal refresh-session lifetime.

### Rolling refresh behaviour

An active user remains signed in: when their access JWT is near expiry, the client uses the refresh flow; the server rotates the refresh token and extends the **idle** expiry from the latest valid activity. If no valid activity/refresh occurs for 30 days, the refresh session expires and the user must authenticate again.

The absolute expiry is intentionally not extended indefinitely. At 90 days, a fresh primary authentication is required even for an active user. This limits the lifetime of a stolen long-lived session. A project may adjust these durations in its PRD, but cannot remove both idle and absolute expiry without an explicit risk ADR.

## JWT access token

1. **MUST** sign access tokens with an approved algorithm and validate an allow-listed algorithm; never accept an algorithm chosen by an untrusted token header.
2. **MUST** validate signature, expiry, issuer, audience, token type, and required subject/session claims.
3. **MUST** include only minimal non-sensitive claims: subject/user ID, session ID, issuer, audience, issued/expiry time, token ID, and authentication time as needed.
4. **MUST NOT** place passwords, secrets, refresh tokens, personally sensitive profile data, or the full permission set in an access JWT.
5. **MUST** calculate permissions, tenant scope, relationship scope, account status, and business-state authorization on the server for protected actions. A JWT proves authentication; it is not the final authorization decision.
6. **MUST** use HTTPS only and send a bearer access token in the Authorization header when this model is selected.

## Refresh tokens and rotation

1. **MUST** make refresh tokens opaque, cryptographically random, and single-use after rotation.
2. **MUST** store only a one-way hash of each refresh token in the database; the raw value is returned only once to the client.
3. **MUST** rotate the refresh token on every successful refresh: revoke/replace the old token and issue a new token in the same session family.
4. **MUST** treat reuse of an already rotated/revoked refresh token as suspected compromise: revoke that session family, record a security audit event, and require login again.
5. **MUST** invalidate the server-side refresh session on logout, password change/reset, account disable/suspension, or administrator security action.
6. **MUST** support “logout this device” and “logout all devices.”

When Redis is used, its session-helper key TTL is set to the current server-calculated idle expiry and is renewed only after a valid refresh/activity path. MySQL still enforces both the idle and absolute expiry.

## Browser and mobile token handling

1. **MUST NOT** store refresh tokens in `localStorage`, `sessionStorage`, URLs, browser history, logs, or analytics events.
2. **SHOULD** send the refresh token as a `Secure`, `HttpOnly`, appropriately `SameSite` cookie with a narrow refresh-path scope for same-site web applications.
3. **MUST** implement CSRF protection when browser cookies authenticate refresh or protected requests.
4. **SHOULD** keep the access JWT in application memory, not durable browser storage; on page reload, obtain a new access token through the secure refresh flow.
5. **MUST** use a platform-appropriate protected credential store for a future native mobile application; do not copy browser storage rules blindly to native apps.

## Multi-device sessions

Each successful login creates a distinct `user_session` record. A user can view/revoke their sessions. Store only privacy-minimized device context, such as a label/user agent summary, last-used time, creation time, and coarse IP/security metadata according to retention policy.

## Session record

```text
user_sessions
  id, user_id, active_tenant_id (if applicable), active_organization_id (if applicable), session_family_id,
  refresh_token_hash, created_at, last_used_at,
  idle_expires_at, absolute_expires_at,
  rotated_at, revoked_at, revoked_reason,
  device_label, user_agent_summary, source_ip (retention controlled),
  created_by_user_id, updated_by_user_id
```

`session_family_id` connects token rotations for one device/login. Do not persist the raw access or refresh token.

For multi-tenant products, the selected tenant context is an explicit session/token claim and is verified against current server-side membership on every protected request. Switching tenant creates a new active context/token; the client may not substitute another tenant ID.

## Required audit events

- login success/failure
- refresh success/failure and refresh-token reuse detection
- logout current/all/device session
- session expiry/revocation
- password reset/change
- email verification/change
- account status change
- SSO login/link/unlink

Never put raw session IDs, access tokens, refresh tokens, or passwords in audit or technical logs.
