# Workflow Template

> Use for a critical, multi-step journey such as registration, booking, approval, payment, import, export or account recovery. A workflow exposes state changes, permissions and failures before implementation.

## Workflow overview

| Field | Value |
| --- | --- |
| Workflow ID | `WF-[module]-[number]` |
| Name | `[name]` |
| PRD / stories | `[links]` |
| Trigger | `[user action, event, schedule, integration]` |
| Primary actor | `[role]` |
| Tenant scope | `[single tenant / approved platform action]` |
| Success outcome | `[observable outcome]` |
| Owner | `[role/name]` |

## Preconditions and invariants

- Authentication/session: `[requirement]`
- Authorization: `[permission(s)]`
- Required state/data: `[state]`
- Invariants that must never break: `[e.g., one tenant only; no duplicate confirmed booking]`
- Concurrency/idempotency rule: `[how retry and simultaneous requests behave]`

## Main flow

| Step | Actor/system | Action | Validation / business rule | State/data change | User/API response | Audit/event |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | `[actor]` | `[action]` | `[rule]` | `[change]` | `[response]` | `[record/event]` |

## Alternate and failure flows

| ID | Condition | Safe behaviour | Message / response | Retry, compensation or escalation |
| --- | --- | --- | --- | --- |
| AF-01 | `[condition]` | `[behaviour]` | `[safe user/client feedback]` | `[rule]` |

## Boundaries and hand-offs

| Boundary | Requirement |
| --- | --- |
| Frontend | `[loading, validation, accessible feedback, duplicate-action prevention]` |
| API | `[endpoint/event, request/response, idempotency]` |
| Database | `[transaction, consistency, indexes/audit]` |
| Integration | `[timeout, retry, outbox/compensation, ownership]` |
| Operations | `[metric, log, alert/runbook requirement]` |

## Verification

- [ ] Happy path, each alternate flow and each permission boundary has acceptance criteria.
- [ ] Cross-tenant and unauthorized access attempts are tested.
- [ ] State transitions and side effects are idempotent or intentionally protected.
- [ ] Failure cannot silently lose or duplicate critical data.
