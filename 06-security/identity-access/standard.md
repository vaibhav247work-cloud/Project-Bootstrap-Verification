# Identity and Access Standard

## Policy decision

The default authorization model is RBAC (Role-Based Access Control).

```text
user -> one or more roles -> one or more permissions
```

A user may hold multiple roles. Their effective permissions are the union of the active permissions granted by those roles.

Example: Ramesh has `teacher` and `staff` roles, so he receives the permissions of both.

## Permission-first, not screen-first

1. **MUST** define permissions as stable resource/action codes, for example `attendance.mark`, `employee.update`, or `report.export`.
2. **MUST NOT** make frontend screens, routes, or menu names the source of authorization truth.
3. **MUST** enforce permission and data-scope checks on every protected backend request.
4. **SHOULD** use the same effective permissions in the frontend to show or hide modules, menus, buttons, and actions.
5. **MUST** deny access by default when no explicit permission grants it.
6. **MUST** use the Dynamic Permission Catalog Standard for permission registration and new-feature behaviour.

## Roles

1. **MUST** make roles reusable bundles of permissions, not hard-coded `if user is admin` checks throughout the product.
2. **MUST** support many-to-many user-role assignment: one user can have many roles and one role can belong to many users.
3. **SHOULD** ship system roles such as `organization_admin`; project-specific roles are created from the product's job functions.
4. **MUST** make a role's scope explicit: system-wide, organization/tenant, or a lower business scope.
5. **MUST** audit creation, editing, activation/deactivation, permission changes, and user assignment/removal of privileged roles.

## Data scope

Permission answers *what action* can be taken. Scope answers *on which records*.

| Example role | Permission | Scope |
|---|---|---|
| Teacher | `attendance.mark` | Classes assigned to the teacher |
| HR manager | `employee.view` | Own organization |
| Platform admin | `organization.manage` | All organizations |

1. **MUST** evaluate tenant/organization ownership as part of every protected data request.
2. **MUST** use relationship/attribute checks when RBAC alone is too broad, such as checking whether a teacher is assigned to the requested class.
3. **MUST NOT** trust a record ID, organization ID, role, or scope sent by the frontend without server-side verification.

## Individual permission overrides

Direct user permissions are not part of normal access management. They are allowed only as a controlled exception.

1. **MUST** require a business reason, grantor/approver identity, creation time, and expiry time.
2. **MUST** be limited to the smallest required permission and scope.
3. **MUST** expire automatically; an expired override must grant no access.
4. **MUST** be visible in an access-review report and recorded in an immutable audit log.
5. **MUST NOT** use a direct override to silently give broad administrator access.
6. **SHOULD** require separate approval for sensitive permissions such as user/role management, exports, finance, and security configuration.

See `dynamic-permissions.md` for code-first registration and administrator inheritance rules.

## User lifecycle and authentication

The same user identity model supports registration, administrator-created accounts, invitations, SSO, imports, and integrations. The project chooses which entry points are enabled.

Recommended user statuses: `invited`, `active`, `suspended`, `disabled`.

1. **MUST** prevent non-active user accounts from receiving authenticated access.
2. **MUST** link an SSO identity to the same internal user record; SSO changes how a user authenticates, not how RBAC authorizes them.
3. **MUST** define whether first SSO login creates a user automatically (just-in-time provisioning) or requires an existing invitation/assignment.
4. **MUST** audit login success/failure, SSO linking, status changes, role changes, and overrides without storing credentials/tokens in logs.

See `authentication-policy.md` for the default email identity and PRD-selected account-creation/SSO policy.

## Access review

1. **MUST** provide a way to answer: "What access does this user effectively have, and why?"
2. **MUST** provide a way to answer: "Which users have this sensitive permission?"
3. **SHOULD** review privileged roles and active overrides periodically; the frequency is decided per project risk.
