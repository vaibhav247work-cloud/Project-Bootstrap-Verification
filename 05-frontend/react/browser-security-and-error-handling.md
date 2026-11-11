# React Browser Security, Privacy and Error-Handling Standard

## Security boundary

The browser is an untrusted client. React UI protections reduce risk, but Spring Boot remains the authority for identity, permissions, tenant/object scope, validation and sensitive data access.

## XSS-safe rendering

- React's normal JSX text rendering is the default. Never interpolate untrusted values into raw HTML, inline script, inline event handlers, CSS text, `javascript:` URLs or DOM sinks such as `innerHTML`.
- `dangerouslySetInnerHTML` is forbidden unless a documented PRD use case needs trusted rich content. It then requires a reviewed allow-list sanitizer, a constrained HTML subset, tests for hostile payloads, and an ADR naming the owner.
- Markdown, rich text, external content, SVG, URL query values and API data are untrusted until parsed/sanitized for their exact display context.
- User-provided URLs must use an allow-list of schemes and, where appropriate, allowed hosts. Never use a URL directly for navigation, iframe, image/embed or download merely because it arrived from an API.
- Do not load third-party scripts/widgets, tag managers, chat tools, fonts or analytics without security/privacy review, an owner, purpose, allowed origin, CSP update, consent requirement and removal/review date.

## Browser storage, credentials and privacy

- Access JWT stays in memory; opaque refresh token stays only in the secure HttpOnly cookie model defined by the authentication policy. Neither goes to `localStorage`, `sessionStorage`, IndexedDB, URL, clipboard, logs, telemetry or error reports.
- `localStorage`, IndexedDB and Cache Storage may contain only explicitly approved low-risk preferences or offline data with owner, tenant/user scope, version, TTL/size cap and logout/tenant-switch deletion policy.
- Do not persist passwords, recovery codes, payment data, identity documents, health data, full profiles or unapproved personal data in browser storage.
- Use `Cache-Control`/response policy from backend/deployment to prevent confidential pages/API responses being stored in shared browser/proxy caches. Never cache sensitive authenticated responses in a service worker by accident.
- Any analytics, session replay, APM or crash reporting follows data-minimisation, consent and retention rules. It must mask fields, URLs/query values and headers that can contain credentials or personal data.

## Platform security headers and origin controls

These are deployed by the web host/reverse proxy and verified by frontend delivery; they are not a React-only concern.

- Enforce HTTPS and use a production Content Security Policy (CSP). Start/report in a controlled mode, remediate violations, then enforce. Use strict allow-lists/nonces or hashes as architecture permits; avoid `unsafe-inline` and broadly trusted origins.
- CSP explicitly constrains scripts, styles, images, fonts, API connections, workers, frames, media and form destinations. The allowed `connect-src` origins include only approved APIs/realtime endpoints.
- Prevent clickjacking with `frame-ancestors` CSP (or the required equivalent). If embedding is a product feature, explicitly allow only approved parents and test it.
- Set the appropriate `Referrer-Policy`, `Permissions-Policy`, MIME-sniffing protection and other environment headers through the deployment security standard. Document exceptions in an ADR.
- CORS is a backend configuration, not a browser bypass: use exact approved origins and never depend on a frontend setting to make an insecure CORS policy safe.
- Environment variables embedded in a web build are public. They may contain public API base URLs or non-secret identifiers, never passwords, private keys, client secrets, database credentials or admin tokens.

## Error-boundary architecture

Use error boundaries at two levels:

```text
app root boundary        -> safe full-app recovery page
route/feature boundary   -> isolated feature failure + retry/back navigation
```

- A rendering error must not leave a blank white screen. Show a safe, accessible fallback with retry, support/trace reference where appropriate, and an unaffected navigation/recovery path.
- Error boundaries do not catch every failure: event-handler, async request, server-rendering or browser-global errors require their own controlled handling. Feature mutation/query layers handle API errors; global handlers record last-resort failures safely.
- Do not display stack traces, raw Problem Details bodies, API URLs, token claims, tenant database details, internal IDs or third-party error content to end users.
- Error recovery must not repeat a non-idempotent action automatically. Reset/retry only the failing view/query after user intent or a safe rule.
- Test fallback UI, route recovery, reset navigation and failure of lazy-loaded chunks.

## Error normalization and user communication

The platform HTTP client converts errors to a small safe shape:

```ts
type AppError = {
  category: "network" | "authentication" | "authorization" | "validation" |
            "conflict" | "rate_limit" | "dependency" | "unexpected";
  status?: number;
  code?: string;        // stable safe backend code only
  traceId?: string;
  retryAfter?: number;
};
```

Features map this to clear actions without exposing technical detail. Validation remains near the field/form; global failures use accessible status/fallback UI. Error messages never reveal whether a hidden record, account, permission or tenant exists.

## Logging, telemetry and diagnostics

- Define a telemetry event schema with approved event name, purpose, fields, retention, sampling, destination and owner. Default to the minimum data needed.
- Attach correlation/trace IDs received from trusted server responses when useful. Do not invent sensitive user identifiers in telemetry.
- Before reporting client exceptions, redact `Authorization`, cookies, tokens, password/reset/OTP fields, request bodies, private URLs/query values and personal data. Disable or scrub automatic console/network capture that violates this.
- Source maps for production are uploaded privately to approved monitoring only; they are not publicly served when that would expose source unnecessarily.
- `console.log` debugging is removed or guarded from production builds. A production diagnostic must use the approved safe telemetry path.
- Monitor error rate, chunk-load failures, API category, route/version/browser and Core Web Vitals, avoiding high-cardinality/sensitive labels.

## Supply chain and build safety

- Add dependencies only for a clear product need. Prefer maintained, small, trusted packages; pin/lock versions and remove unused dependencies.
- CI runs dependency vulnerability/license review under the project policy. High/critical findings are remediated or have an explicit time-bound risk acceptance.
- Third-party package code, remotely loaded configuration and browser extensions are not trusted simply because TypeScript compiles.
- Builds are reproducible from lockfile, use least-privilege CI tokens, and never place production secrets in build logs/artifacts.

## Required tests/checks

- Hostile text/URL/rich-content cases; no unsafe HTML/URL sink without approved sanitizer
- Storage inspection after login/logout/tenant switch; no credential persistence
- CSP enforced/report-only verification, iframe/clickjacking and allowed-origin checks
- Root and route error-boundary fallbacks; chunk/API/error normalization paths
- Telemetry redaction and production source-map access controls
- Dependency and third-party-script inventory review
