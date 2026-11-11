# Tenant Context Model

## Meaning of terms

| Term | Meaning in this model |
|---|---|
| `tenant` | The isolated client/workspace data boundary, for example `xyzschool` or `jhkschool` |
| `organization` | A business organization inside a tenant, for example `abc` |
| `user` | One global login identity, identified by email |
| `membership` | A user's allowed relationship to a tenant/organization, including roles and scope |

The same organization name may exist in more than one tenant. Therefore organization names/codes are **not globally unique**.

```text
tenant xyzschool  -> organization abc
tenant jhkschool  -> organization abc

one user/email
  -> membership in xyzschool / abc
  -> membership in jhkschool / abc
```

## Login and context selection

1. User authenticates once using their global email identity.
2. Backend loads active memberships for that user.
3. If exactly one eligible tenant context exists, select it automatically.
4. If more than one eligible tenant context exists, return only the tenant/organization choices the user is a member of; the user chooses one.
5. Backend creates/updates an authenticated session with the selected active tenant context.
6. Every protected request derives its tenant context from the authenticated server-side session/token and verifies membership. The client cannot access a different tenant merely by sending another ID.

Example: an employee assigned only to `xyzschool` is automatically placed in `xyzschool`; they never see `jhkschool`. A user with memberships in both sees a tenant selector and receives only the selected tenant's data.

## Authorization calculation

```text
Can the user act on a record?
  = user account active
  AND active session tenant matches the record tenant
  AND active membership permits the action
  AND assigned roles grant the permission
  AND organization/workspace/relationship scope permits the record
  AND business state permits the action
```

Roles must be assigned at a membership/scope level, not merely globally on the user. A user can be `teacher` in one tenant and `staff` in another.
