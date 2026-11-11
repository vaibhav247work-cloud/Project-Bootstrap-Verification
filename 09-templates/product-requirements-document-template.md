# Product Requirements Document (PRD) Template

> Use for a new product, module or material feature. Complete only the sections relevant to the change, but do not omit a required decision: write `Decision needed` with an owner and due date instead. Link to supporting documents rather than copying them. This document is approved before solution design starts.

## 1. Document control

| Field | Value |
| --- | --- |
| Product / feature | `[name]` |
| Client / business unit | `[name]` |
| Author | `[name]` |
| Product owner | `[name]` |
| Technical owner | `[name]` |
| Status | `Draft / In review / Approved / Superseded` |
| Version / last updated | `[version, date]` |
| Target release | `[date or milestone]` |
| Linked decisions | `[ADR, discovery notes, contracts]` |

## 2. Problem, outcome and boundaries

### Problem statement

`[Who has which problem today? What evidence supports it?]`

### Desired outcome and success measures

| Outcome | Measure | Baseline | Target | Measurement owner |
| --- | --- | --- | --- | --- |
| `[outcome]` | `[metric]` | `[current value / unknown]` | `[target]` | `[role]` |

### Scope

- In scope: `[capabilities/workflows]`
- Explicitly out of scope: `[items]`
- Assumptions and constraints: `[budget, dates, policy, existing system, device/network constraints]`

## 3. Users, actors and access

| Actor/persona | Goal | Authentication method | Roles/permissions needed | Tenant/access boundary |
| --- | --- | --- | --- | --- |
| `[e.g., teacher]` | `[goal]` | `[email/password, SSO, passkey]` | `[permission names or decision needed]` | `[tenant rule]` |

State whether account creation is self-registration, invite, administrator provisioning, SSO just-in-time provisioning, or another approved method. State the primary login identifier and whether SSO/passkey is required, optional or out of scope.

## 4. Workflows and functional requirements

Describe the important happy path, failure path and permission boundary. Number requirements so they can be linked to designs, tests and releases.

| ID | Priority | User need / rule | Acceptance outcome | Owner |
| --- | --- | --- | --- | --- |
| FR-01 | `Must` | `As a [actor], I need [capability], so that [outcome].` | `[observable result]` | `[role]` |

For each critical workflow, add:

### Workflow: `[name]`

1. `[Actor starts with ...]`
2. `[System validates ...]`
3. `[System changes/returns ...]`
4. `[Failure/permission/duplicate/retry behaviour ...]`
5. `[Audit/notification/integration behaviour ...]`

Link detailed user stories or wireframes here: `[link]`.

## 5. Business rules and configuration

| Rule ID | Rule | Owner | Configurable? | Validation / exception |
| --- | --- | --- | --- | --- |
| BR-01 | `[rule]` | `[business owner]` | `Yes / No` | `[how enforced; who may override]` |

For configurable rules, define default value, allowed range/schema, effective date, tenant scope and audit expectation. Do not use free-form executable scripts for business configuration; use versioned, validated structured configuration.

## 6. Data and integrations

### Data

| Data domain | Create/read/update/delete actors | Retention / deletion need | Sensitivity | Reporting/search need |
| --- | --- | --- | --- | --- |
| `[e.g., booking]` | `[roles]` | `[rule or decision needed]` | `[public/internal/personal/sensitive]` | `[queries/exports]` |

Record import/export, migration, attachment and audit needs. Confirm whether the feature is in the tenant's isolated database and whether any cross-tenant operation is prohibited or explicitly approved.

### Integrations

| System | Purpose | Direction | Data shared | Auth / owner | Failure and retry behaviour |
| --- | --- | --- | --- | --- | --- |
| `[provider]` | `[why]` | `In / Out / Both` | `[minimum fields]` | `[method / owner]` | `[safe behaviour]` |

## 7. Non-functional requirements

| Area | Requirement / decision | How it will be verified |
| --- | --- | --- |
| Performance | `[critical journey, expected usage, response target]` | `[test/metric]` |
| Scale | `[tenant/user/record/traffic estimate and growth]` | `[load/capacity test]` |
| Availability | `[support hours, uptime/recovery need]` | `[SLO/DR test]` |
| Accessibility | `[required standard and key assistive flows]` | `[manual/automated check]` |
| Devices/network | `[web/mobile/browser/offline constraints]` | `[test matrix]` |
| Localization | `[languages, timezone, currency, date rules]` | `[acceptance test]` |

## 8. Security, privacy and compliance

- Data classification and sensitive fields: `[answer]`
- Required roles/permissions and emergency direct-grant need: `[answer]`
- Audit events: `[create/update/delete/access/export/configuration]`
- Authentication and session requirement: `[answer]`
- Consent, retention, legal/regulatory or client policy: `[answer]`
- Threats or abuse cases requiring design review: `[answer]`

Link the threat model and security review when applicable. Backend authorization remains authoritative; hiding a frontend control is not access control.

## 9. Delivery, operations and support

- Feature flag / safe rollout requirement: `[answer]`
- Migration / backfill / tenant sequencing: `[answer]`
- Monitoring, alerts and success metric dashboard: `[answer]`
- Backup/restore or recovery implication: `[answer]`
- Support owner, support hours and escalation/contact path: `[answer]`
- Required runbook, release notes and client communication: `[answer]`

## 10. Risks, dependencies and open decisions

| ID | Type | Description | Impact | Owner | Due date | Status |
| --- | --- | --- | --- | --- | --- | --- |
| D-01 | `Decision / risk / dependency` | `[description]` | `[impact]` | `[person/role]` | `[date]` | `Open` |

## 11. Approval to design

Approval means the scope, outcome, actors, critical workflows, data/security constraints and unresolved material decisions are understood. It does not approve an unreviewed technical architecture or release.

| Role | Name | Decision | Date |
| --- | --- | --- | --- |
| Product/business owner | `[name]` | `Approved / changes needed` | `[date]` |
| Technical owner | `[name]` | `Approved / changes needed` | `[date]` |
| Security/privacy owner, when applicable | `[name]` | `Approved / changes needed` | `[date]` |
