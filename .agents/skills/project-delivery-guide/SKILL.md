---
name: project-delivery-guide
description: Guide a new product or material feature from intake through implementation readiness, playbook review, and risk-based verification. Use when a user starts, sets up, designs, implements, reviews, or verifies a project governed by this engineering playbook.
---

# Project Delivery Guide

Turn this playbook into a guided delivery workflow. Keep all product-specific documents, code, secrets, credentials, customer data, and implementation details in the client repository, never in this playbook repository.

## Choose the entry point

- **New product or module:** begin with discovery and project start.
- **New material feature:** begin with feature readiness; require its approved PRD requirement, workflow, and acceptance criteria.
- **Existing project review:** inspect only the relevant artifacts and report unmet checklist items, risks, and evidence gaps.
- **Implementation or verification:** confirm the required design decisions and choose tests based on the actual change risk.

Do not make the user repeat information already supplied. Do not ask a broad questionnaire at once. Ask the next smallest group of questions that unlocks the current stage. If the answer is missing but does not materially affect scope, security, data ownership, tenant isolation, cost, or external effects, state a reasonable assumption and record it as such.

## Stage 1 — Discovery and project start

Read these sources before creating or assessing the project-start artifacts:

- `10-checklists/project-start-and-feature-ready.md`
- `09-templates/product-requirements-document-template.md`
- `01-discovery/project-brief.md` when the user has only an early idea
- `00-project-start/universal-standards.md`

Collect or record: named owners and support escalation, problem and measurable outcome, scope/out-of-scope, actors, critical workflows, tenant model, authentication/account creation, roles and data scope, data classification/retention, integrations, scale and accessibility needs, and material unknowns with owner and due date.

Create the PRD in the client repository from the PRD template. Keep its status as Draft until the required decisions are understood; do not imply approval. For an early idea, create a project brief first, then progress to the PRD.

## Stage 2 — Design gate

Before material coding, require the applicable workflow and acceptance criteria, plus data, API, access, and security decisions. Read only the matching templates and standards:

- Workflow and acceptance: `09-templates/workflow-template.md`, `09-templates/acceptance-criteria-template.md`, `09-templates/user-story-template.md`
- Architecture and decisions: `09-templates/solution-architecture-template.md`, `02-architecture/default-technology-decisions.md`, `02-architecture/adr-and-governance/adr-template.md`
- Data: `09-templates/data-model-and-erd-template.md`, `03-databases/mysql/`
- API: `09-templates/api-contract-template.md`, `04-backend/spring-boot/api-standard.md`
- Access: `09-templates/access-control-design-template.md`, `06-security/identity-access/`
- Threat/security/privacy: `09-templates/threat-model-template.md`, `09-templates/security-and-privacy-review-template.md`

Default to a modular monolith with React + TypeScript, Spring Boot, MySQL, REST + OpenAPI, and responsive web. Record a reason for a material deviation. Tenant databases remain isolated; no cross-tenant access or client-selected datasource is permitted without explicit approval.

Use backend-enforced RBAC. A direct permission is exceptional, time-bound, approved, and audited. For each protected workflow, make the permission name, trusted tenant rule, data scope, denial behavior, audit behavior, and allow/deny tests explicit.

## Stage 3 — Implementation readiness and delivery

Use `10-checklists/project-start-and-feature-ready.md` to check every feature-ready item. Do not silently skip a sub-step: mark it complete, deferred with reason, or blocked with the decision owner.

When implementing, preserve the playbook's requirements for versioned migrations, DTO/API contracts, API-boundary validation, bounded queries/pagination, audit needs, no secrets in code/logs, and backend files of 800 lines or fewer. Use the relevant frontend, backend, database, security, and deployment standard only when the task touches that area.

Before any external mutation—deploying, provisioning paid services, sending messages, modifying shared production-like systems, or using credentials—prepare the change and ask for explicit confirmation. Never put credentials, tokens, customer data, or internal security details in generated documents or reports.

## Stage 4 — Verification and handoff

Read before selecting verification:

- `09-templates/test-strategy-and-evidence-template.md`
- `07-testing/quality-strategy.md`
- the relevant files under `07-testing/`
- `10-checklists/code-review-and-security-review.md` for review work
- `10-checklists/release-and-deployment.md` and `10-checklists/post-release-and-handover.md` for release/handoff work

Create or update a test strategy linked to requirements and workflows. Select the minimum evidence by risk, not by test count. At a minimum, protected or material workflows need appropriate authorization, tenant-isolation, validation/failure, duplicate/retry, and audit/side-effect coverage. Use synthetic or approved masked data only.

Run the project’s relevant existing checks where available: formatting/linting, type checking, unit tests, integration tests, contract tests, build, and any affected E2E/security/migration checks. Do not claim a result passed without reproducible evidence. If an environment, dependency, credential, or approval is missing, say exactly what was not verified and why.

## Required response format

At each meaningful checkpoint, report:

1. **Status:** current stage and whether it is ready to progress.
2. **Completed:** artifacts or implementation completed.
3. **Decisions needed:** only material open questions, each with its effect.
4. **Deferred/blocked:** item, reason, owner, and due date if known.
5. **Verification:** checks run, results, and evidence locations; distinguish unrun checks.
6. **Next action:** the smallest useful next step.

Keep updates concise. Never present a draft, assumption, or unrun test as approved, confirmed, or passed.
