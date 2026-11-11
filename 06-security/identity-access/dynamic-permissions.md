# Dynamic Permission Catalog Standard

## Goal

A new feature must not require an administrator to manually create its permission in the database. The application registers a permission catalog from version-controlled code during deployment/startup. Access grants remain controlled.

```text
Feature code declares permission
        ↓
Deployment syncs it into the permission catalog
        ↓
Backend protects API with it
        ↓
Frontend uses the same code for UI visibility
        ↓
Role/access policy decides who receives it
```

## Non-negotiable safety rule

**Automatic registration is allowed. Automatic broad access is not.**

A newly registered permission is denied to custom roles until an authorized administrator assigns it. This preserves least privilege.

## Permission format

Use lowercase, dot-separated codes:

```text
<module>.<resource>.<action>
```

Examples:

```text
attendance.record.view
attendance.record.mark
employee.profile.update
employee.profile.export
booking.reservation.cancel
access.role.manage
```

Rules:

1. **MUST** make permission codes stable API/business concepts; do not use route paths or visual screen names.
2. **MUST** keep the same permission when a screen is renamed, redesigned, or split.
3. **MUST** create a new permission when the action or sensitivity meaningfully changes.
4. **MUST** classify each permission with `module`, `scope_level` (`platform` or `organization`), `risk_level`, and active/deprecated status.
5. **MUST NOT** reuse a retired code for a different meaning.

## Catalog declaration

Every backend module owns a small, version-controlled catalog declaration containing:

```text
code: employee.profile.export
name: Export employee profiles
module: employees
scope_level: organization
risk_level: high
default_assignment: none
```

At startup/deployment, the permission-sync process:

1. creates a missing permission;
2. updates safe metadata such as display name/description;
3. never silently deletes an existing permission or role assignment;
4. marks removed code declarations as deprecated, then removes them only through a reviewed migration/ADR;
5. writes a deployment audit event describing catalog changes.

The sync must be idempotent: running it again produces the same result.

## Dynamic access policy by role kind

| Role kind | New permission behaviour |
|---|---|
| `platform_admin` | Automatically effective for all platform and organization permissions, within platform authority |
| `organization_admin` | Automatically effective for all organization-scoped permissions in that organization; no platform authority |
| `module_admin` | Automatically effective only for permissions in its explicitly assigned module(s) and organization scope |
| `custom_role` | Denied until explicitly assigned by an authorized administrator |

Do not implement administrator power by copying every permission into their role at each release. Use an explicit role-kind/policy evaluated by the authorization layer. This lets new permissions work predictably while preserving their scope boundaries.

## Frontend and backend contract

1. **MUST** protect every sensitive backend operation with the required permission, even if the screen is hidden.
2. **SHOULD** expose an authenticated `my-effective-permissions` endpoint or token claim sufficient for the frontend to render allowed UI.
3. **MUST** derive backend authorization from the server-side catalog, not from client-supplied permission values.
4. **MUST** add a test that fails when a protected API references an unregistered permission.
5. **SHOULD** add a test that all protected frontend routes/actions declare a known permission.

## Admin UI behaviour

The role editor reads the catalog dynamically and groups permissions by module/resource/action. It shows:

- role kind and scope;
- effective permissions, including inherited dynamic permissions;
- explicit grants for custom roles;
- active temporary overrides, their reason, approver, and expiry;
- warning before assigning high-risk permissions.

## New-feature release checklist

- Permission code and metadata declared in backend catalog
- API/action protected and tested server-side
- UI mapped to the same permission
- Scope and organization checks implemented
- Default policy chosen: admin inheritance or custom-role deny
- Audit event defined if the action is sensitive
- Documentation and role-editor display verified
