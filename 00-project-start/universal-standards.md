# Universal Mandatory Standards

These rules apply to every product and stack.

## Product and design

1. **MUST** define user roles, data ownership, and acceptance criteria before implementing a protected feature.
2. **MUST** build the smallest valuable release first; defer features without a clear user outcome.
3. **MUST** record major technical decisions in an ADR.
4. **SHOULD** use a modular monolith first. Split into services only for a demonstrated scaling, ownership, security-boundary, or deployment need.

## Security

1. **MUST** enforce authorization on the server for every protected action and record; UI checks are only usability.
2. **MUST** validate all input at API boundaries, allow-list values where practical, and use parameterized database access.
3. **MUST NOT** store passwords, API keys, tokens, or production secrets in source code, commits, client apps, or logs.
4. **MUST** use HTTPS in every non-local environment and secure cookie/token handling.
5. **MUST** hash passwords using a modern password-hashing algorithm; never encrypt or store them as plain text.
6. **MUST** implement rate limiting or equivalent abuse protection for login, password reset, public forms, and expensive endpoints.
7. **MUST** log security-relevant events without logging secrets or unnecessary personal data.
8. **SHOULD** use OWASP ASVS as the security-verification baseline; choose the verification level based on product risk.

## Data and reliability

1. **MUST** manage schema changes through reviewed, versioned migrations; manual production schema edits are prohibited.
2. **MUST** define backup frequency, retention, restore ownership, and restore testing for production data.
3. **MUST** maintain referential integrity and business constraints in the database where possible.
4. **MUST** define deletion behaviour: hard delete, soft delete, archive, or anonymization. Never leave it implicit.
5. **MUST** add an audit trail for financial, approval, permission, status, and sensitive-data changes when the product needs accountability.
6. **MUST** record who created and last updated a user-managed transactional record; use a documented system actor for automated changes.

## APIs and integrations

1. **MUST** publish an API contract (OpenAPI for REST APIs) before or alongside implementation.
2. **MUST** use consistent error responses, validation errors, pagination, date/time format, and API-version policy.
3. **MUST** make externally retried write operations idempotent when duplicate execution could create harm.
4. **SHOULD** use asynchronous jobs/queues for slow or retryable work such as email, exports, and large imports.

## Quality and operations

1. **MUST** have automated tests for critical business rules, permissions, and failure paths.
2. **MUST** use code review and an automated build before merge.
3. **MUST** keep development, staging, and production configuration/data separate.
4. **MUST** externalize deploy-specific configuration; do not hard-code it.
5. **MUST** provide structured logs, health checks, error tracking, and basic production monitoring.
6. **MUST** define a rollback or recovery path before a production release.
