# API Contract, End-to-End and Cross-Tenant Testing Standard

## API contract is a tested product boundary

The version-controlled OpenAPI document is the contract for every exposed HTTP API. It declares paths, methods, authentication, permission/data-scope expectations, tenant context, request/response schema, pagination, errors, idempotency, rate limits and deprecation.

Every provider test verifies the running Spring Boot API matches the published contract. Every consumer test verifies that frontend/integration code correctly handles the contract it depends on.

## Required contract tests

- Request validation: required/optional fields, type/range/format, unknown-field policy, multipart/file limits and safe error mapping.
- Response validation: success, empty result, pagination/cursor, locale/time/currency representation, nullability, error Problem Details and relevant headers.
- Authentication/authorization: 401, 403 and 404/non-disclosure behaviour; required permission and data-scope contract.
- Commands: idempotency-key requirement/duplicate replay behaviour, conflict response, audit requirement and retry-safe semantics.
- List endpoints: allow-listed filters/sorts, maximum page size, cursor integrity and no data leak through filter/search.
- Versioning: old supported client compatibility, deprecation header/documentation, and replacement/sunset behaviour.
- Events/webhooks: schema version, signature/replay/idempotency, correlation/tenant association and retry contract.

## Compatibility gate

The CI compares a proposed OpenAPI contract to the released/supported baseline. Breaking changes require a new version or approved migration/deprecation plan.

Breaking examples include removing/renaming an endpoint/field, changing type/format/requiredness/meaning, narrowing accepted enum/input, changing error/status semantics, removing a supported auth flow, or changing pagination/token interpretation. Adding an optional field or optional endpoint is normally compatible but still requires client resilience tests.

Generated frontend types/clients are regenerated and type-checked whenever the contract changes. Generated code does not remove the need for runtime error, unexpected-field and backward-compatibility behaviour tests.

## End-to-end test scope

E2E tests run against a deployed, production-like stack: React build, Spring Boot, two tenant databases, Redis and required controlled dependencies. They validate a small set of critical journeys rather than every validation permutation.

Each test:

- creates/uses isolated seeded tenant, user, role, permission and domain data;
- starts from clean browser/session/storage state;
- performs the user-visible workflow through UI and verifies outcome through supported UI/API/observability evidence;
- cleans only its owned test data or uses disposable environment/database;
- captures screenshots, trace, browser console/network metadata and server correlation IDs on failure, with secrets/PII redacted.

Use accessible role/label-based locators or intentional `data-testid` contracts when user-visible selection is insufficient. Do not rely on CSS classes, DOM nesting, sleep delays or another test's data/session.

## Critical end-to-end journeys

Every product selects its own PRD journeys. At minimum, when applicable, test:

1. Login/session bootstrap, refresh near expiry, logout and post-logout access rejection.
2. A normal permitted user views/searches/paginates permitted data and completes one core write flow.
3. A denied user cannot discover action, cannot use deep link and receives server rejection if the request is attempted.
4. Role/permission/temporary override change takes effect after defined invalidation/refresh and produces required audit evidence.
5. An idempotent critical command (booking/payment/create) survives a duplicate click/retry without duplicate business result.
6. Validation, business conflict, network failure and safe recovery preserve user intent where policy allows.
7. A sensitive admin/configuration workflow applies correct permission, re-authentication/audit and error behaviour.

## Mandatory cross-tenant isolation matrix

Run these with at least `tenant_a` and `tenant_b`, including matching record IDs and similar user/profile names where helpful to expose accidental collision:

| Scenario | Expected result |
| --- | --- |
| Tenant A authenticated user reads A record | A record only |
| Tenant A user requests B record ID/direct URL | deny/not found without confirming B record exists |
| Tenant A user searches/lists/filter/sorts | never sees B metadata/counts/suggestions |
| Tenant A cached response then B request with same record ID | B result only; no cache collision |
| Session/logout/tenant switch | old tenant context/data/cache cleared before new context display |
| Job/outbox/webhook/file URL generated for A | executes/accesses only A context; B cannot consume it |
| Tenant A datasource unavailable | controlled A failure; B remains isolated/healthy |
| Wrong/missing/forged tenant client value | server rejects/safely resolves trusted context; no default/fallback tenant |

Repeat critical isolation scenarios at API integration and deployed E2E layers. A frontend-only test is insufficient because datasource/cache/background paths are server-owned.

## Test environments and accounts

- Use separate non-production tenant databases and explicit test identities; names/codes clearly indicate test purpose.
- CI secrets are least-privilege and scoped to test environments. Browser traces/screenshots/logs are protected artifacts with retention limits.
- No E2E test runs dangerous commands against production. Production smoke tests, if approved, are read-only or use a dedicated reversible test tenant with explicit operational approval.
- External payment/email/SMS/SSO/etc. use controlled sandbox/emulator/test doubles, never real recipients or money without approved test process.

## Execution policy

- Fast contract/schema and targeted E2E tests run for affected pull requests.
- Full critical journey and tenant-isolation suite runs on main/release candidate and before production release.
- Contract incompatibility, tenant boundary failure, authentication/authorization regression or unreconciled critical E2E failure blocks release.
- Keep a concise failure artifact and link it to defect/incident; do not resolve by broad retries or weakening the assertion.
