# MySQL Reference Schema: Identity and Access

This is a logical reference model. Adapt names and extra columns to the product, but preserve the relationships and constraints.

```text
tenants --< organizations
users --< user_memberships >-- tenants
users --< user_roles >-- roles --< role_permissions >-- permissions
  |
  `--< user_permission_overrides >-- permissions
  |
  `--< user_sessions
```

## Core tables

| Table | Purpose | Key rules |
|---|---|---|
| `tenants` | Isolated client/workspace boundary | Example: `xyzschool`; a tenant has one or more organizations |
| `organizations` | Organization inside a tenant | Name/code is unique within its tenant, not globally |
| `users` | One global internal identity per human/system account | Status controls login; no plain password/token fields in logs |
| `user_memberships` | User access to a tenant/organization context | Active membership determines tenant chooser/auto-selection |
| `roles` | Reusable job-function bundles | Tenant or system scope and role kind are explicit |
| `permissions` | Version-controlled dynamic catalog | Example: `attendance.record.mark` |
| `user_roles` | User-to-role assignment | Unique active assignment for user, role, and scope |
| `role_permissions` | Role-to-permission mapping | Unique role/permission pair |
| `user_permission_overrides` | Exceptional direct grants only | Reason, approver, start/end time, audit required |
| `user_sessions` | One browser/device login and rotating refresh-token state | Store refresh-token hash only; supports logout/revocation |

## Essential relationships

```text
users.id                         -> user_roles.user_id
tenants.id                       -> organizations.tenant_id
users.id                         -> user_memberships.user_id
tenants.id                       -> user_memberships.tenant_id
roles.id                         -> user_roles.role_id
roles.id                         -> role_permissions.role_id
permissions.id                   -> role_permissions.permission_id
users.id                         -> user_permission_overrides.user_id
permissions.id                   -> user_permission_overrides.permission_id
users.id (grantor/approver)      -> user_permission_overrides.approved_by_user_id
users.id                         -> user_sessions.user_id
```

## Suggested important columns

### `permissions`

```text
id, code, name, module, description, is_active,
scope_level, risk_level, is_deprecated,
created_at, updated_at, created_by_user_id, updated_by_user_id
```

`code` is unique and stable. A screen may use multiple permissions; a permission may protect more than one API/screen.

Catalog records are created/updated by an idempotent deployment sync. Never give application runtime users unrestricted permission-catalog editing rights.

### `roles`

```text
id, code, name, role_kind, organization_id, is_active,
created_at, updated_at, created_by_user_id, updated_by_user_id
```

`role_kind` is one of `custom_role`, `module_admin`, `organization_admin`, or `platform_admin`. The authorization layer applies the controlled dynamic inheritance rules for administrator kinds; custom roles rely only on explicit `role_permissions` records.

### `user_roles`

```text
id, user_id, role_id, organization_id, scope_type, scope_id,
starts_at, ends_at, assigned_by_user_id, created_at, updated_at,
created_by_user_id, updated_by_user_id
```

Use `ends_at` for temporary role assignments. Scope representation must be finalized per project; do not add polymorphic scope columns without a documented need.

### `user_memberships`

```text
id, user_id, tenant_id, organization_id, status,
starts_at, ends_at, created_at, updated_at,
created_by_user_id, updated_by_user_id
```

The combination reflects the actual business membership model. A user with one active eligible tenant context is auto-selected after login; multiple active contexts are returned in a tenant chooser. Role assignments must be tied to this membership/scope where roles differ by tenant.

### `user_permission_overrides`

```text
id, user_id, permission_id, organization_id, scope_type, scope_id,
effect, reason, approved_by_user_id, starts_at, expires_at,
revoked_at, revoked_by_user_id, created_at, updated_at,
created_by_user_id, updated_by_user_id
```

Rules:

- `effect` is initially restricted to `grant`; adding explicit `deny` requires a separate ADR because conflict resolution becomes complex.
- `reason`, `approved_by_user_id`, and `expires_at` are mandatory.
- An override is effective only after `starts_at`, before `expires_at`, and while `revoked_at` is null.
- Enforce `expires_at > starts_at` with application validation and a database `CHECK` where supported by the deployed MySQL version.
- Do not create a permanent override; convert recurring access into an appropriately scoped role.

## Effective-access calculation

```text
effective permissions
  = active role permissions
  + active, unexpired user permission grants

authorization decision
  = user status is active
  AND permission exists
  AND tenant/organization check passes
  AND relationship/scope rule passes
  AND business-state rule passes
```

The backend performs this calculation or uses a safe cached representation. The frontend may read it for UI visibility only.

See `../../06-security/identity-access/token-session-policy.md` for session fields, refresh-token rotation, expiry, revocation, and multi-device rules.
