# MySQL Multi-Tenancy Standard

## First decide the actual isolation model

Do not add `organization_id` or `tenant_id` just because a generic playbook mentions it. The PRD/architecture must first define whether the product is:

1. **Single organization**: one organization in one deployment/database. No organization/tenant scope column is needed on normal business tables.
2. **Tenant contains one or more organizations**: a shared deployment/database serves several independent tenant boundaries. A tenant ID is required on tenant-owned data; organization names are unique only within their tenant.
3. **One organization has multiple tenant/workspace contexts**: model the real business boundary explicitly (for example `tenant_id` plus `organization_id`, or `workspace_id`), only where records genuinely belong to that context.
4. **Separate deployment/database per customer/organization**: cross-customer scope column is normally unnecessary; isolation is provided by deployment/database boundaries.

Use clear project terminology. `tenant`, `organization`, `customer`, and `workspace` are not interchangeable labels; each must have a defined meaning.

## Classification

Every table **MUST** be classified before implementation when a product has more than one data boundary:

| Class | Examples | Rule |
|---|---|---|
| Platform-global | permission catalog, countries | No `organization_id`; platform-admin only changes |
| Tenant/workspace-owned | employees, bookings, invoices | Tenant scope ID required in shared database/schema and authorization-scoped |
| Organization-owned within tenant | organization settings, departments, custom workflow | Tenant + organization scope as the real model requires |
| Cross-tenant platform record | platform support/audit metadata | Must state why cross-tenant visibility is allowed |

## Required rules

1. **MUST** add a non-null scope ID (`organization_id`, `tenant_id`, or `workspace_id`, whichever the model defines) only to records stored in a shared database/schema that genuinely belong to that scope.
2. **MUST** enforce the defined scope in backend reads and writes; a client-supplied scope ID is never trusted by itself.
3. **MUST** include scope in foreign-key and uniqueness design only where it prevents a real cross-scope mistake.
4. **MUST** use the active scope context from the authenticated server-side session/request when scope isolation applies.
5. **MUST** test cross-scope isolation only for products that share scope data in one deployment/database.
6. **MUST NOT** query scoped data without an explicit scope policy, except for audited platform-administration/reporting paths.
7. **SHOULD** include the scope ID in indexes only when actual queries filter by it and the query plan supports it.

## Tenant + organization pattern

When a tenant can contain organizations, use this relationship:

```text
tenants 1 -- * organizations
users 1 -- * user_memberships * -- 1 tenants
```

An organization identifier must be unique within its tenant, for example `UNIQUE (tenant_id, code)`. Do not create a global uniqueness constraint on an organization name/code that may legitimately repeat in another tenant.

The active tenant context is selected after login from the user's memberships. All tenant-owned data queries are scoped to it. See `../../02-architecture/tenant-context-model.md`.

## Tenant lifecycle

Define onboarding, suspension, data retention, export, offboarding, and restore requirements before production. Tenant deletion is a business/legal decision; it must not be implemented as an accidental cascade.
