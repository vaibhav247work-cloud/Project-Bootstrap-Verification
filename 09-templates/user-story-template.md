# User Story Template

> One story describes a small, independently understandable user outcome. It links back to an approved PRD requirement and does not hide a large workflow behind a vague title.

## Story card

| Field | Value |
| --- | --- |
| Story ID | `US-[module]-[number]` |
| Title | `[short outcome, not implementation]` |
| PRD requirement | `[FR-xx link]` |
| Module / tenant scope | `[module and scope]` |
| Priority | `Must / Should / Could / Won't now` |
| Status | `Draft / Ready / In progress / In review / Done` |
| Product owner | `[role/name]` |
| Delivery owner | `[role/name]` |

## User outcome

> As a `[persona/role]`, I want to `[action/capability]`, so that `[measurable value]`.

## Context and rules

- Entry point / trigger: `[screen, API, event or schedule]`
- Preconditions: `[login, permission, existing data, configuration]`
- Business rules: `[BR-xx links or written rule]`
- Permission required: `[module.resource.action]`
- Data created/changed/read: `[entities and audit expectation]`
- Integration or notification: `[if any]`
- Explicitly not included: `[boundary]`

## Behaviour examples

| Scenario | Given | When | Then |
| --- | --- | --- | --- |
| Happy path | `[state]` | `[actor action]` | `[observable outcome]` |
| Validation | `[invalid/missing state]` | `[action]` | `[clear safe feedback; no unintended change]` |
| Authorization | `[actor lacks permission]` | `[attempt]` | `[backend denies; no data exposed/changed]` |
| Duplicate/retry | `[prior request/state]` | `[repeat action]` | `[idempotent or clearly handled result]` |
| Failure | `[dependency/system problem]` | `[action]` | `[safe outcome, error and retry/escalation behaviour]` |

## Ready checklist

- [ ] User, value and scope are clear.
- [ ] Permission and tenant boundary are stated.
- [ ] Business rules, validation and failure behaviour are known or tracked as decisions.
- [ ] UX/accessibility, API/data and test impacts are identified.
- [ ] Dependencies and acceptance criteria are linked.

## Done checklist

- [ ] Acceptance criteria pass in the intended environment.
- [ ] Backend authorization and tenant isolation are verified.
- [ ] Required audit, observability, error handling and documentation are complete.
- [ ] Automated tests and release evidence link to this story.
- [ ] Product owner accepts the observable outcome.
