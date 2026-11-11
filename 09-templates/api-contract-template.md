# API Contract Template

> Create a version-controlled OpenAPI contract before or alongside implementation. This template captures the decisions that must accompany the schema. Follow the backend API standard in [`04-backend/spring-boot/api-standard.md`](../04-backend/spring-boot/api-standard.md).

## Contract metadata

| Field | Value |
| --- | --- |
| API / module | `[name]` |
| Version | `/api/v1` |
| OpenAPI source | `[repository/path/link]` |
| Owner | `[role/team]` |
| Consumers | `[web/mobile/integration]` |
| Authentication | `[JWT/session/service credential]` |
| Tenant context source | `[trusted server-side mechanism]` |
| Deprecation/review date | `[date/condition]` |

## Operation catalogue

| Operation ID | Method/path | Purpose | Required permission | Input | Success | Errors | Idempotency/rate limit |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `[module.action]` | `POST /api/v1/[resources]` | `[purpose]` | `[module.resource.action]` | `[schema]` | `[201 + response]` | `[Problem Details]` | `[key/limit]` |

## Request and response rules

- DTOs only; never expose persistence entities or accept arbitrary maps.
- Define required fields, formats, length/range/enums, defaults, nullability and cross-field validation in OpenAPI.
- Use ISO-8601 UTC timestamps and explicit monetary/decimal representation.
- For lists, state allowed filter/sort fields, bounded page size and cursor/offset choice.
- Use the shared Problem Details error contract; errors are safe, stable and do not expose secrets, tenant data or stack traces.
- State whether create/command requests require an idempotency key and how key reuse behaves.

## Endpoint behaviour details

### `[operation ID]`

| Concern | Decision |
| --- | --- |
| Authorization and data scope | `[backend permission + tenant rule]` |
| Validation | `[field/business validation]` |
| State change/transaction | `[entities/events]` |
| Audit event | `[event/actor]` |
| Retry/duplicate handling | `[rule]` |
| Timeout/dependency behaviour | `[safe response/retry]` |
| Observability | `[correlation ID, metric/log]` |
| Compatibility | `[additive/deprecation/client impact]` |

## Change and verification record

| Change | Compatibility | Consumer notification | Contract/API/integration test | Approval |
| --- | --- | --- | --- | --- |
| `[change]` | `Compatible / breaking` | `[plan]` | `[link]` | `[owner/date]` |

Breaking changes require a documented deprecation path, replacement and sunset date; they are not silently released under an existing contract version.
