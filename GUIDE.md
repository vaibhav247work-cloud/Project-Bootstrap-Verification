# Engineering Playbook: How to Use It

This playbook is the reusable standard. Keep it stable. For every client product, create project-specific documents in that product repository (for example, `docs/product/`) and copy only the relevant templates there.

## The short workflow

1. Understand the client problem and complete the PRD.
2. Turn approved requirements into workflows, stories and acceptance criteria.
3. Design architecture, data, API, permissions and security before material coding.
4. Implement module by module using the backend, frontend and database standards.
5. Verify with the relevant test, security and performance rules.
6. Release through the operational readiness, rollout and rollback process.
7. Verify production and hand over support/documentation.

Do not skip a stage because the product sounds simple. A small feature may need a short version of a document, but it still needs the relevant decisions.

## Guided project agent

This repository includes a guided project agent. In Codex, start with either of these prompts:

```text
Use $project-delivery-guide to start a new product. Ask the first material questions only.
```

```text
Use $project-delivery-guide to review this project for implementation readiness and run the relevant checks.
```

The agent asks only for decisions that materially affect the product, security, data ownership, tenant isolation, cost, or an external action. It records ordinary assumptions, creates project-specific documents in the client repository, and reports checks that were run, deferred, or blocked. It does not deploy, send messages, provision paid services, or use credentials without your explicit confirmation.

## 1. Start a new client/product

Create a product workspace such as:

```text
client-product/
  docs/
    product/
  backend/
  frontend/
```

Copy these files into `docs/product/`:

- [`09-templates/product-requirements-document-template.md`](09-templates/product-requirements-document-template.md)
- [`10-checklists/project-start-and-feature-ready.md`](10-checklists/project-start-and-feature-ready.md)

Use the project-start checklist to gather the client decisions. Important questions include:

- What problem, user outcome and success measure matter?
- Who will use it, and which roles/access scopes exist?
- Is self-registration, invite, admin provisioning, SSO or passkey needed?
- Which data is sensitive and how long must it be retained?
- What integrations, reports, imports/exports and notifications are required?
- What scale, support hours, availability, language, device and accessibility needs exist?

An unknown is acceptable only when it is recorded as `Decision needed` with an owner and due date.

## 2. Complete the PRD before design

Use the PRD to write the problem, outcome, scope, actors, business rules, integrations, non-functional needs and risks. Ask the client for approval of the scope before beginning technical design.

For a very small change, the PRD may be only a few pages, but it must still state the user, expected behaviour, access, data and success condition.

## 3. Turn requirements into buildable work

For every important workflow, create:

- a [user story](09-templates/user-story-template.md);
- a [workflow](09-templates/workflow-template.md); and
- [acceptance criteria](09-templates/acceptance-criteria-template.md).

Cover the happy path plus validation, lack of permission, duplicate/retry and dependency failure. The acceptance criteria must be observable and testable.

## 4. Make the design decisions

Use only the templates needed by the feature:

| Need | Template / standard |
| --- | --- |
| Module/component decision | [Solution architecture](09-templates/solution-architecture-template.md) and [ADR](02-architecture/adr-and-governance/adr-template.md) |
| Tables, relations, indexes, JSON configuration | [Data model and ERD](09-templates/data-model-and-erd-template.md) and [`03-databases/`](03-databases/) |
| REST endpoints and OpenAPI | [API contract](09-templates/api-contract-template.md) and [`04-backend/spring-boot/api-standard.md`](04-backend/spring-boot/api-standard.md) |
| Roles, permissions and exceptional access | [Access-control design](09-templates/access-control-design-template.md) |
| Sensitive data, login, integration or abuse risk | [Threat model](09-templates/threat-model-template.md) and [security/privacy review](09-templates/security-and-privacy-review-template.md) |

Default product decisions already established by this playbook:

- React + TypeScript for responsive web; choose native/mobile packaging only when the PRD proves the need.
- Spring Boot, module-first structure, Hibernate/JPA and global error handling.
- MySQL by default; select PostgreSQL only when its requirements justify it.
- One isolated database per tenant. No unapproved cross-tenant data path.
- RBAC is backend-enforced. Users may have multiple roles. Direct permission grants are exceptional, time-bound, approved and audited.
- Email is the default login identifier. SSO and passkeys are requirement-driven.

## 5. Implement safely

Read the relevant standards before coding:

- database: [`03-databases/`](03-databases/)
- backend: [`04-backend/`](04-backend/)
- frontend: [`05-frontend/`](05-frontend/)
- security: [`06-security/`](06-security/)

Build one module or coherent feature at a time. Do not create a large generic framework before the first real module proves its need. Keep each backend source file at 800 lines or fewer; split by responsibility.

Use frontend permission checks for navigation and UX only. Every protected backend route and data operation must independently verify permission and tenant scope.

## 6. Verify before release

Use the [test strategy and evidence template](09-templates/test-strategy-and-evidence-template.md), then follow [`07-testing/`](07-testing/). At minimum, verify critical workflows, authorization, tenant isolation, validation, error paths and migrations.

Before production, complete:

- [security review](10-checklists/code-review-and-security-review.md);
- [release readiness](10-checklists/release-and-deployment.md); and
- the relevant documents in [`08-devops-deployment/`](08-devops-deployment/).

## 7. Release and operate

Use the [release/operational readiness template](09-templates/release-and-operational-readiness-template.md). Confirm immutable artifact, database migration plan, feature flag/wave plan, rollback path, monitoring, runbooks and support contacts before deployment.

After release, run the post-release checklist. If an incident occurs, follow [`08-devops-deployment/incident-response-runbooks-and-oncall.md`](08-devops-deployment/incident-response-runbooks-and-oncall.md), then record learning and actions with the [incident/post-incident/handover template](09-templates/incident-postincident-and-handover-template.md).

## Efficient Codex prompts

Use this pattern. Replace only the bracketed items:

```text
Use the engineering playbook at [path].

Goal: [exact result].
Relevant context: [only the relevant docs/files/paths].
Constraints: [stack, module, rules that matter].
Done when: [tests/documents/behaviour that prove completion].

Ask only questions that materially block a decision. For minor details, state a reasonable assumption.
Keep the response under [number] bullets. Do not change files outside [scope].
```

Examples:

```text
Use the engineering playbook. Goal: design the booking module.
Read only the PRD, MySQL rules, Spring API standard and RBAC design template.
Create data model, API contract and permission catalogue. Do not code yet.
Stop after listing material decisions that need approval. Keep the response under 10 bullets.
```

```text
Implement approved booking module only. Follow the existing module-first Spring structure.
Change only booking files and related migrations/OpenAPI/tests. Run targeted booking tests.
Done when acceptance criteria AC-01 through AC-08 pass. Return changed files, test result and open risks only.
```

## Reduce credit/token use

- Start a chat for one coherent outcome, not an entire client product forever.
- Name exact paths instead of pasting long documents or attaching the whole repository.
- State desired output length, for example `max 10 bullets` or `final result only`.
- Ask for targeted research only when current information matters; request a source limit, e.g. `maximum 3 official sources`.
- Batch clearly related steps: `complete template steps 1-3, then stop for approval` instead of many vague `next` messages.
- Keep durable rules in the short `AGENTS.md`; do not repeat them in every prompt.
- Ask to run targeted tests, not every test suite, unless the change has broad impact.
- Use `/compact` when a chat is long, or open a new chat and provide the PRD/design file paths.

## Improve this playbook

After each project, update a global standard only when a rule is reusable across future products. Keep client-specific decisions, credentials, data, diagrams and implementation details inside that client product repository, not here.
