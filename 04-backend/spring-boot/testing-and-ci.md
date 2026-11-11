# Backend Testing and CI Quality Gate

## Required tests

- Unit tests for business rules/value objects/policies
- Integration tests against MySQL, Redis and security configuration for critical flows
- API/contract tests for OpenAPI and error responses
- Authorization tests: allow/deny, tenant isolation, ownership and state transitions
- Authentication tests: login, refresh rotation, revocation, password reset, passkey/SSO when enabled
- Migration/rollback and database concurrency tests
- HTTP-client retry/timeout/circuit-breaker and job retry/idempotency tests

## Merge/release gate

1. **MUST** compile, format, static-analyze, run relevant tests and generate/update OpenAPI in CI before merge.
2. **MUST** fail CI for dependency vulnerabilities at the project-defined severity threshold; exceptions require ADR/expiry.
3. **MUST** enforce the 800-line production-source-file hard limit.
4. **MUST** run integration tests with containerized/real compatible dependencies, not only mocks/in-memory DBs.
5. **MUST** run security/tenant/critical-flow regression tests before release.
6. **SHOULD** use coverage as a signal for critical code, not a vanity percentage; every bug fix gets a regression test when practical.
