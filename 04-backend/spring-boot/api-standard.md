# API Standard

## Contract first

Every public/internal API is described in version-controlled OpenAPI before or alongside implementation. The contract records authentication, permission, tenant behaviour, request/response schemas, pagination, errors, idempotency, rate-limit and deprecation policy.

## URI and response rules

1. **MUST** use `/api/v1/...` for versioned REST APIs and plural resource nouns: `/bookings`, `/employees`.
2. **MUST** use correct HTTP verbs/statuses; `POST` create/action, `GET` read, `PATCH` partial update, `PUT` full replacement only where truly supported, `DELETE` defined deletion action.
3. **MUST** use request/response DTOs; do not expose entities or accept arbitrary JSON maps.
4. **MUST** use ISO-8601 UTC timestamps and explicit currency/decimal JSON representation.
5. **MUST** implement bounded pagination, allowed sort fields and filter allow-lists. Default/maximum sizes are documented per endpoint.
6. **MUST** use a stable Problem Details error contract from global error handling.
7. **MUST** require an idempotency key for client-retriable create/payment/command endpoints where duplicate processing causes harm.
8. **MUST** deprecate before breaking/removing an externally used API; document replacement and sunset date.

## List response

```json
{
  "items": [],
  "page": { "size": 50, "nextCursor": "opaque-value" }
}
```

Cursor pagination is preferred for high-volume/changing lists. Offset pagination is permitted for small/admin datasets with a bounded maximum.

## API ownership

Each endpoint declares module owner, required permission, tenant context source, data scope, audit requirement, rate limit, and OpenAPI operation ID.
