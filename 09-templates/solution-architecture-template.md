# Solution Architecture Template

> Complete after the PRD is approved and before material implementation. This records the smallest solution that meets the requirement. Link an ADR for every consequential or reversible-later decision; do not turn this document into a diagram-only description.

## 1. Context and decision summary

| Field | Value |
| --- | --- |
| Product / feature | `[name]` |
| PRD / workflow | `[links]` |
| Technical owner | `[role/name]` |
| Status | `Draft / Reviewed / Approved / Superseded` |
| Architecture/ADR links | `[links]` |
| Last reviewed | `[date]` |

### Problem and constraints

- Outcome and critical workflows: `[summary/link]`
- Existing systems and required reuse: `[answer]`
- Tenant model: `One isolated database per tenant; no cross-tenant query unless an explicitly approved platform capability exists.`
- Scale, availability, security or compliance constraints: `[answer]`
- Deliberately excluded complexity: `[answer]`

## 2. Components and boundaries

| Component | Responsibility | Owner/module | Inputs/outputs | Data owned | Trust boundary |
| --- | --- | --- | --- | --- | --- |
| `[React client]` | `[responsibility]` | `[module]` | `[API/events]` | `None / local UI state` | `[browser boundary]` |
| `[Spring Boot module]` | `[responsibility]` | `[module]` | `[API/events]` | `[domain data]` | `[service boundary]` |
| `[MySQL tenant DB]` | `[persistence]` | `[owner]` | `[queries/migrations]` | `[tables]` | `[tenant database]` |
| `[Redis/provider]` | `[purpose]` | `[owner]` | `[keys/API]` | `[ephemeral data]` | `[network/provider]` |

Draw or link a diagram showing user/client, frontend, backend modules, tenant datasource, Redis, asynchronous processing and external providers. Label every network and trust boundary.

## 3. Critical request and event flows

For each critical workflow, document sequence, auth/permission check, tenant selection, transaction boundary, side effects, retry/idempotency and failure/compensation behaviour.

| Flow | Entry | Authorization and tenant rule | Sync/async steps | Failure / retry / compensation | Observability |
| --- | --- | --- | --- | --- | --- |
| `[e.g., confirm booking]` | `[screen/API/event]` | `[permission + trusted tenant]` | `[steps]` | `[safe outcome]` | `[metric/log/trace/audit]` |

## 4. Data and consistency

- System of record for each domain: `[answer]`
- Database migration and backward compatibility approach: `[answer]`
- Transaction boundary and concurrency strategy: `[answer]`
- Events/outbox/jobs and delivery/idempotency guarantee: `[answer]`
- Search/cache/read model and invalidation rule: `[answer]`
- Retention, backup/restore and deletion implications: `[answer]`

Link the data model and API contract. Do not store authorization truth only in the frontend or cache.

## 5. Security and operational design

| Area | Design decision | Verification / evidence |
| --- | --- | --- |
| Authentication/session | `[decision]` | `[test/config]` |
| Authorization | `[RBAC/direct-grant rule]` | `[backend tests]` |
| Secrets/credentials | `[approved secret source]` | `[deployment evidence]` |
| Sensitive data | `[classification/minimization/encryption]` | `[review/test]` |
| Resilience | `[timeouts, retries, circuit breaker, fallback]` | `[test/runbook]` |
| Observability | `[logs, metrics, traces, audit]` | `[dashboard/alert]` |
| Rollout/recovery | `[flag, migration, rollback, restore]` | `[runbook]` |

## 6. Alternatives, risks and approval

| Decision/risk | Options considered | Chosen approach and why | Trigger to revisit | Owner |
| --- | --- | --- | --- | --- |
| `[item]` | `[options]` | `[decision]` | `[metric/date/event]` | `[role]` |

Use the canonical [ADR template](../02-architecture/adr-and-governance/adr-template.md) for each significant decision. Approvals: product `[name/date]`, technical `[name/date]`, security/privacy where applicable `[name/date]`.
