# Engineering Playbook Agent Instructions

## Purpose

Use this repository as the master enterprise product-engineering playbook. Preserve reusable standards. Put client-specific PRDs, credentials, data and implementation details in that client's repository, not here.

## Work method

1. Read only the folders/templates relevant to the requested product stage.
2. Start from PRD and project-start checklist for a new product.
3. Before material coding, require the relevant workflow/acceptance criteria and design decisions for data, API, access and security.
4. Ask a concise question only if a missing decision materially changes scope, security, data ownership, tenant isolation, cost or external effect. Otherwise state a reasonable assumption.
5. Never silently skip a listed phase sub-step. State what is deferred and why.
6. Keep explanations concise unless the user asks to learn the concept.
7. For a new product, feature setup, implementation readiness, project review, or verification request, use the `project-delivery-guide` skill in `.agents/skills/project-delivery-guide/`.

## Non-negotiable rules

- Default stack: React + TypeScript, Spring Boot, MySQL. Deviate only when requirements justify it.
- One isolated database per tenant. Do not introduce unapproved cross-tenant access or client-controlled datasource selection.
- Backend authorization is authoritative. Use RBAC; multiple user roles are allowed. Direct permissions are only exceptional, time-bound, approved and audited.
- Do not expose secrets, tokens, personal data, tenant data, stack traces or internal security details in client responses, logs or documents.
- Use versioned migrations, DTO/API contracts, input validation, bounded queries/pagination and audit requirements as defined by the standards.
- Keep backend source files at 800 lines or fewer; split by responsibility.
- Do not weaken testing, security, tenant isolation, migration safety, rollout/rollback or incident-operation standards for speed.

## Efficient use

- Prefer exact file paths and targeted reads/tests over broad repository scans.
- Use web research only when information is current, security/legal/financial, or explicitly requested; prefer authoritative sources and keep sources limited.
- Match response length to the request. Return changed files, verification and material open risks; avoid repeating documents already available in the repository.
- For changes, edit the smallest correct scope and verify in proportion to risk.
