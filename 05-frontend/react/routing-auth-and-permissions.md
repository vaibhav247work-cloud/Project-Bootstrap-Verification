# Routing, Authentication and Permission-Aware UI Standard

Frontend route/permission checks are UX only. Spring Boot always authenticates, validates tenant/object scope and performs permission/business-state authorization again.

Each central route registry entry declares path, loader, authentication need, required permission, tenant-context need, metadata and error boundary. Public routes are explicit. All product routes require authentication by default. Validate route parameters and all return URLs; only approved internal return paths are allowed.

## Session lifecycle

Default browser model: short-lived access JWT in memory, opaque refresh token in a `Secure`, `HttpOnly`, appropriate `SameSite` cookie. Refresh/access tokens never go into browser storage, URLs, logs or analytics.

```text
startup -> bounded loading -> refresh (with CSRF where required)
-> access token in memory -> current-user/effective-access summary -> protected app
```

One platform session/API client owns token attach, refresh deduplication and one retry. On refresh failure, logout, disable or invalid tenant context: clear in-memory identity and user/tenant cache and navigate to login. A 401 means reauthenticate; a 403 is access denied, not automatic logout. Logout first revokes session on server.

## Permissions and tenant

Use typed stable permission codes, e.g. `can("booking.reservation.cancel")`; never primary-check role names such as teacher/staff. New navigation comes from a typed feature registry containing route, label, permission and optional flag. Hidden actions improve discoverability only; direct routes/API stay protected. UI handles permission changes and expired overrides by invalidating access state.

The browser never chooses an arbitrary tenant ID header/query. In the current DB-per-tenant model, server/session establishes the context. A future tenant selector must get allowed choices from server, switch through a validated server endpoint, then evict tenant query/offline cache and reload. Every tenant-scoped cache key carries approved tenant identity.

Test public/auth/denied/not-found routes, deep-link redirect safety, bootstrap/expiry/logout, 401 vs 403, permission UI and tenant cache isolation. Never expose credentials or sensitive payloads to telemetry.
