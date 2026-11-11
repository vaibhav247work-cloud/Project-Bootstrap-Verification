# Test Strategy and Quality Evidence Template

> Write this for a product, release or material feature before implementation is complete. Link automated reports and UAT evidence; do not claim a test passed without reproducible evidence.

## Scope and risk

| Field | Value |
| --- | --- |
| Feature/release | `[name]` |
| PRD/stories/workflows | `[links]` |
| Quality owner | `[role]` |
| Critical risks | `[data loss, tenancy, payment, auth, etc.]` |
| Exit decision owner | `[role]` |

## Risk-based coverage

| Requirement/workflow | Risk | Unit | Integration | API/contract | E2E/UAT | Performance/security | Evidence link |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `[FR/flow]` | `High/Med/Low` | `[test]` | `[test]` | `[test]` | `[test]` | `[test]` | `[link]` |

Mandatory for material protected workflows: authorization, cross-tenant isolation, validation/failure, duplicate/retry behaviour and audit/side effect verification.

## Test data and environment

- Environment/version/configuration: `[answer]`
- Test tenant/data source: `[synthetic/approved masked data only]`
- External dependency approach: `[sandbox/mock/contract test]`
- Non-functional workload/acceptance target: `[answer]`
- Known limits/exceptions and approval: `[answer]`

## Exit evidence

| Gate | Result | Evidence | Owner | Exception/expiry |
| --- | --- | --- | --- | --- |
| Required tests passed | `[pass/fail]` | `[link]` | `[role]` | `[link]` |
| Critical defects resolved/accepted | `[result]` | `[link]` | `[role]` | `[link]` |
| Security/performance evidence reviewed | `[result]` | `[link]` | `[role]` | `[link]` |
| UAT accepted where required | `[result]` | `[link]` | `[role]` | `[link]` |
