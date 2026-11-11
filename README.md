# Engineering Playbook

This repository is a reusable decision system for building secure, maintainable software products. It applies to ERP, booking, employee-management, workflow, and SaaS products.

## Default baseline

- Frontend: React + TypeScript, responsive-first
- Backend: Java + Spring Boot
- Database: MySQL
- API: REST with an OpenAPI contract
- Architecture: modular monolith by default

Other choices are allowed when a documented requirement calls for them. Record important exceptions in an Architecture Decision Record (ADR).

## How to use for a new product

1. Complete `01-discovery/project-brief.md`.
2. Complete `06-security/security-and-access-matrix.md`.
3. Create ADRs in `02-architecture/adr-and-governance/` for decisions that differ from the default baseline.
4. Apply every rule marked **MUST** before release.
5. Add only the relevant technology playbooks as the project needs them.

## Rule levels

- **MUST**: mandatory; an exception needs an ADR and owner approval.
- **SHOULD**: default practice; document why it is not used.
- **MAY**: optional, requirement-driven.

## Directory map

- `00-project-start`: cross-cutting universal rules and playbook entry point.
- `01-discovery`: client discovery, product scope, users, flows, and acceptance criteria.
- `02-architecture`: decision guides, ADRs, integration boundaries, and reusable modules.
- `03-databases`: vendor-specific database rules.
- `04-backend`: backend-language/framework rules.
- `05-frontend`: frontend rules.
- `06-security`: security baseline, identity/access, privacy, and threat modelling.
- `07-testing`: test strategy, test levels, quality gates, and test templates.
- `08-devops-deployment`: environments, CI/CD, monitoring, backups, release/rollback.
- `09-templates`: reusable PRD, ADR, API, test, and release templates.
- `10-checklists`: project-start, feature, security, release, and production checklists.

Governance and reusable-module specifications live under `02-architecture` rather than as extra root folders.
