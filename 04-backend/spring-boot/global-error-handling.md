# Spring Boot Global Error Handling Standard

## One global boundary

Use one application-wide `@RestControllerAdvice` to map known application exceptions to a consistent Problem Details JSON response. Modules throw meaningful typed exceptions; they do not create their own inconsistent HTTP error payloads.

```text
module application/domain exception
        ↓
global exception handler
        ↓
safe HTTP status + Problem Details JSON + trace ID
```

## Exception categories

| Category | Typical status | Public response rule |
|---|---:|---|
| Request validation/type error | 400 | Field error codes/messages safe for the caller |
| Unauthenticated/invalid session | 401 | Generic authentication failure |
| Permission/scope denied | 403 | Do not reveal hidden resources or permissions |
| Record not found | 404 | Do not leak cross-tenant existence |
| Business conflict/version conflict | 409 | Safe conflict code, e.g. booking unavailable |
| Rate limited | 429 | Include safe retry guidance where applicable |
| External dependency failure | 502/503/504 | Generic dependency-unavailable message |
| Unexpected exception | 500 | Generic message only; full cause remains internal |

## Rules

1. **MUST** include a trace/request ID in every error response and structured log entry.
2. **MUST NOT** return stack traces, SQL errors, datasource names, tenant database names, secrets, tokens, internal class names, or raw external-provider responses to clients.
3. **MUST** log unexpected exceptions once at the global boundary with safe contextual fields: trace ID, authenticated user/session ID where safe, tenant code, route, status, exception class, and root cause.
4. **MUST** keep validation errors field-specific but never echo password, token, secret, or sensitive request values.
5. **MUST** map security exceptions through Spring Security `AuthenticationEntryPoint`/`AccessDeniedHandler` into the same response contract.
6. **MUST** write audit events for security-relevant failures where required, separately from technical exception logs.
7. **MUST NOT** catch `Exception` in controllers/services merely to return a response; handle a known exception locally only when a real recovery path exists.

## Response shape

```json
{
  "type": "https://api.example.com/problems/booking-unavailable",
  "title": "Booking cannot be completed",
  "status": 409,
  "code": "BOOKING_UNAVAILABLE",
  "traceId": "01J..."
}
```

`code` is a stable machine-readable application code. `title` is safe, user-facing text. Debug details remain in protected logs.
