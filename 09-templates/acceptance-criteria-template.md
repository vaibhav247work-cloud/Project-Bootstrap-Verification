# Acceptance Criteria Template

> Acceptance criteria describe observable behaviour, not a preferred implementation. They are written with the story/workflow and become the shared basis for development, QA and product acceptance.

## Criteria set

| Field | Value |
| --- | --- |
| Related story/workflow | `[US-xx / WF-xx]` |
| Feature/version | `[name]` |
| Owner | `[role/name]` |
| Test evidence location | `[link]` |

## Criteria

| ID | Priority | Given | When | Then | Evidence |
| --- | --- | --- | --- | --- | --- |
| AC-01 | `Must` | `[precondition]` | `[action/event]` | `[observable result]` | `[test/case/link]` |

Include, where applicable:

- happy path and correct saved/returned state;
- validation, required fields, boundary values and accessible error feedback;
- unauthorized and cross-tenant attempts, verified by the backend;
- duplicate submission, retry, concurrency and idempotency behaviour;
- network, dependency or timeout failure and safe recovery;
- audit/event/notification expectation;
- responsive keyboard, touch and screen-reader-relevant behaviour for critical UI; and
- performance or scale expectation for critical journeys.

## Non-negotiable quality criteria

- [ ] No sensitive data, tenant data or internal stack details are exposed in errors.
- [ ] A user without the permission cannot read or mutate protected data through a direct API call.
- [ ] Success and failure leave data in a valid, explainable state.
- [ ] Criteria can be proven by a test, reviewable record or agreed manual UAT step.

## Acceptance decision

| Decision | By | Date | Notes / remaining approved exception |
| --- | --- | --- | --- |
| `Accepted / changes needed / accepted with time-bound exception` | `[name/role]` | `[date]` | `[link to exception]` |
