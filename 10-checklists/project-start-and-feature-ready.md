# Project Start and Feature-Ready Checklist

## Project start

- [ ] Client/business owner, product owner, technical owner and support/escalation contacts are named.
- [ ] PRD has problem, measurable outcome, scope/out-of-scope, actors and critical workflows.
- [ ] Tenant model is confirmed; default is one isolated database per tenant with no unapproved cross-tenant operation.
- [ ] Login/account creation, SSO/passkey needs and primary identifier are decided or tracked.
- [ ] Roles, permissions, data scope and emergency direct-grant rule are designed.
- [ ] Data classification, retention, integrations, exports/imports and audit needs are known.
- [ ] Scale, availability, device/browser, accessibility, localization and support expectations are known.
- [ ] Material unknowns have owner and due date; no assumption is silently treated as fact.

## Feature ready for development

- [ ] Approved PRD requirement, user story, workflow and acceptance criteria are linked.
- [ ] Permission name and backend tenant/data-scope rule are explicit.
- [ ] Business rules, invalid/duplicate/retry/failure behaviour are defined.
- [ ] API contract, data-model/migration and architecture/ADR impacts are reviewed where needed.
- [ ] Threat/security/privacy impact has been triaged.
- [ ] Test strategy, test data/environment and UAT owner are known.
- [ ] Rollout, migration, observability, runbook and rollback needs are identified.
