# Cross-Stack Quality Strategy and Test Layers

## Goal

Quality means the product correctly serves its intended users under normal and adverse conditions, protects data/access, remains usable, and can be safely changed. Passing a test count or code-coverage number alone is not quality.

Every product starts with a risk-based quality plan linked to its PRD, architecture decisions, permissions, tenant model and acceptance criteria.

## Required quality plan per project

Before feature implementation, record:

```text
critical user journeys and acceptance criteria
data sensitivity and security/compliance requirements
tenant/isolation model and roles/scopes
business invariants and irreversible/financial actions
supported browsers, devices, locales and accessibility target
availability, performance, scale and recovery expectations
external systems, failure/timeout/retry behaviour
test environments, test data ownership, release/rollback owner
```

The plan specifies what is automated, what needs manual/exploratory testing, what evidence is retained, and who accepts any residual risk. It changes with the product; it is not a generic document copied without review.

## Test portfolio

Use a balanced portfolio: many fast deterministic tests, sufficient real integration tests, and a small but powerful critical E2E suite. Avoid both extremes—only unit tests and only slow UI tests.

| Layer | Main purpose | Default owner | Example |
| --- | --- | --- | --- |
| Unit | Pure logic and edge cases | developer | pricing rule, mapper, permission evaluator |
| Component/module | Behaviour at module boundary | developer | React form, Spring service/controller slice |
| Integration | Real collaboration with owned infrastructure | developer | Spring + MySQL migration/JPA + Redis |
| Contract | Consumer/provider compatibility | API owner/consumer | OpenAPI DTO/error/idempotency compatibility |
| E2E | Critical user journey through deployed stack | team/QA | login, role grant, booking, logout |
| Security/resilience | Attack/failure safety | developers + security/QA | tenant escape, refresh reuse, Redis outage |
| Performance/recovery | Service/user experience under expected stress | team/DevOps | 1M-record list, restore, rollout rollback |
| UAT/exploratory | Business fitness and unknown usability defects | product/client/QA | approved scenario acceptance |

Tests run at the lowest layer that proves the behaviour. Duplicate the same assertion across all layers only when the risk justifies defense in depth.

## Non-negotiable test properties

1. Tests are deterministic, isolated, repeatable and readable. They do not depend on execution order, current clock, internet, mutable shared state or production data.
2. Each test owns/creates its data and cleans it safely; parallel test runs cannot read/write one another's tenant, user or record data.
3. Use real MySQL/Redis/containerized dependencies for behaviour that relies on their actual semantics. An in-memory substitute does not prove migration, SQL, locking, index, transaction, TTL or datasource behaviour.
4. External providers are mocked/sandboxed with realistic contract fixtures; never call uncontrolled live providers from normal CI.
5. Test names state business outcome and context, e.g. `staff_cannot_cancel_booking_outside_assigned_location`.
6. Assertions prove outcome, side effect and no forbidden side effect where relevant: response, persisted state, audit/outbox event, cache invalidation, notification/queue call.
7. Tests test user-visible/contract behaviour rather than private implementation details. Refactoring without behavioural change should not require broad test rewrites.

## Risk-based minimum matrix

| Change/risk | Minimum verification |
| --- | --- |
| Pure calculation/formatting | Unit tests incl. boundary values |
| New API/read/write workflow | Unit + service/API integration + contract + affected frontend component |
| Permission, role, scope or temporary override | Allow and deny tests; direct/deep-link/API check; audit/cache invalidation |
| Tenant/database routing | Two-tenant isolation test, wrong-context denial, cache-key/datasource check |
| Schema migration/query/index | Migration on clean + upgraded DB, integration query behaviour, rollback/forward plan |
| Auth/session/password/SSO/passkey | Security scenarios, expiry/reuse/logout/reset and browser/native storage rules |
| Payment/booking/inventory/irreversible command | Idempotency, concurrency/conflict, timeout uncertainty, audit and E2E happy/failure path |
| External API/event/job | Contract fixture, timeout/retry/idempotency/dead-letter behaviour and observability |
| Responsive/accessibility UI | Component/E2E plus keyboard, zoom, screen-reader/manual viewport evidence |
| PWA/offline/real-time/3D | Capability-specific offline/update/fallback/performance/accessibility test |

## Coverage, mutation and defects

- Coverage reports identify blind spots; projects set sensible changed-code/critical-module thresholds but never approve low-value tests solely to raise a percentage.
- Critical calculations, permission evaluators and state machines may use mutation testing or targeted negative tests when risk warrants it.
- Every production defect receives severity, reproduction, root cause, scope, owner and resolution evidence. A defect that could have been automatically detected adds a regression test at the lowest effective layer.
- A flaky test is a defect in the delivery system: quarantine only with issue, owner, expiry and risk assessment. Do not disable it invisibly.

## Ownership and evidence

- Developers own automated tests for their change and must demonstrate relevant quality evidence in review.
- QA/product own acceptance interpretation and exploratory/business testing; this does not transfer engineering quality responsibility away from developers.
- Security/DevOps support specialised verification, production-like environments, observability and recovery testing.
- The release owner confirms all required evidence, exceptions and residual-risk acceptance before release.

Store test plan links, CI artifacts, performance/security reports, UAT sign-off and approved exceptions with the release record. Never store real secrets, raw production PII or credentials in test artifacts.
