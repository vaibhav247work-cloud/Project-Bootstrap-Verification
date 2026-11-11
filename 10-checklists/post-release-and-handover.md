# Post-Release Verification and Operational Handover Checklist

## Post-release verification

- [ ] Critical user workflows and protected API actions work in the production target scope.
- [ ] No unexpected cross-tenant access, auth/session error, migration issue, queue backlog or integration failure is visible.
- [ ] SLO/error/latency and business success signals meet expected range.
- [ ] Logs, traces, audit events and dashboards are receiving safe, useful data.
- [ ] Support/client status and release notes are sent when required.
- [ ] Known defects, follow-up actions and approved exceptions are tracked with owners/dates.

## Operational handover

- [ ] Receiving product/engineering/operations owner is named and accepts ownership.
- [ ] PRD, workflows, ADRs, API contract, data model and permission catalogue are findable.
- [ ] Repositories, build/deploy path, configuration ownership and secret rotation ownership are documented.
- [ ] Dashboards, alerts, SLOs, runbooks, backup/restore proof and on-call route are accessible.
- [ ] Test evidence, UAT decision, release history, known risks and incident follow-ups are handed over.
- [ ] Access is least-privilege; departing/shared accounts are not relied on.
