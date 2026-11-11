# Project Brief Template

Complete this before estimates, design, or coding.

## 1. Outcome

- Product name:
- Business problem:
- Primary users:
- Measurable success metric:
- In scope for first release:
- Explicitly out of scope:

## 2. Users and access

| Role | Main tasks | Data scope | Can approve/change sensitive data? |
|---|---|---|---|
| Example: Organization admin | Manage users and settings | Own organization | Yes |

Define whether the product is single-organization or multi-organization (tenant) before modelling data.

### Authentication decisions

- Primary login identifier: Email (default; change requires ADR)
- Enabled account creation: self-registration / admin-created / invitation / bulk import / SSO JIT / integration API / service account
- Is email verification required?
- Are invitations enabled? If yes, expiry/resend policy:
- Is SSO required? If yes, provider and protocol:
- Is just-in-time SSO user provisioning allowed?
- Who can activate, suspend, disable, and assign initial roles?

### Data-boundary decision

- Deployment model: single organization / shared multi-tenant / separate database per customer
- Define terms used in this project: tenant, organization, customer, workspace
- Which tables actually require a scope ID, if any?
- Can the same email have memberships in multiple tenants? If yes, tenant chooser or auto-selection rules:

## 3. Core flows

For each important flow, record: actor, trigger, happy path, failure/edge cases, permission checks, and acceptance criteria.

## 4. Non-functional requirements

- Expected users and concurrent users:
- Mobile/browser support:
- Availability requirement:
- Performance target:
- Sensitive/personal/financial data:
- Legal, retention, or audit requirement:
- External integrations:
- File uploads/exports/notifications:

## 5. Release definition

A release is ready only when each acceptance criterion, security requirement, migration, critical automated test, monitoring check, and rollback plan is complete.
