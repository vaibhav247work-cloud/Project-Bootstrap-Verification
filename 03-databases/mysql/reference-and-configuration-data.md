# MySQL Reference and Configuration Data Standard

## Purpose

Reference data is controlled business vocabulary. Configuration data changes product behaviour. Treat both as intentional data, not scattered constants or unrestricted text fields.

## Categories

| Type | Examples | Ownership |
|---|---|---|
| Code-managed reference data | permission codes, system roles, internal status definitions | Version-controlled catalog + migration/sync |
| Project seed data | countries, currencies, standard departments | Versioned seed migration |
| Tenant-managed reference data | departments, locations, leave types | Authorized tenant admin UI/API + audit |
| Tenant configuration | branding, policies, workflow thresholds | Tenant admin, validation, audit, safe defaults |

## Rules

1. **MUST** assign ownership and change method to every reference/configuration dataset.
2. **MUST** use a stable code/key in addition to a display name when code, integrations, reporting, permissions, or workflows depend on the value.
3. **MUST** make baseline seed data idempotent and version controlled.
4. **MUST** validate tenant-editable configuration at the backend and audit sensitive changes.
5. **MUST NOT** place secrets, passwords, private keys, or environment-specific deployment configuration in business configuration tables.
6. **MUST NOT** hard-delete a value used by historical transactions; mark inactive/archived and prevent new use when appropriate.
7. **SHOULD** separate feature flags/operational rollout controls from permanent business configuration.

## Status lists

Status definitions are business rules. A project must state whether each status list is code-managed or tenant-configurable. A tenant must never be able to rename or remove a status that backend business logic requires unless the product explicitly supports that workflow configuration.
