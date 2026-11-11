# Access Control and Permission Design Template

> Use when a module introduces screens, APIs, data scopes or administrative capabilities. Permission checks are enforced by the backend. The frontend may improve UX based on permissions but is never the authority.

## 1. Context

| Field | Value |
| --- | --- |
| Product / module | `[name]` |
| PRD / workflows | `[links]` |
| Tenant scope | `[rule]` |
| Owner | `[role/name]` |
| Review date | `[date]` |

## 2. Resource and permission catalogue

Use the standard name: `<module>.<resource>.<action>`.

| Permission | Resource/action | Backend endpoints/use cases | Data scope | Audit needed | Notes |
| --- | --- | --- | --- | --- | --- |
| `booking.booking.create` | `[create booking]` | `[operation IDs]` | `[tenant/own/team]` | `Yes/No` | `[notes]` |

Do not create a permission per visual button without an independently securable action. A newly registered permission is not automatically granted to custom roles; role assignment remains an explicit business decision.

## 3. Role design

| Role | Business purpose | Included permissions | Explicit exclusions | Assignment authority | Default? |
| --- | --- | --- | --- | --- | --- |
| `[teacher]` | `[purpose]` | `[permissions]` | `[exclusions]` | `[admin role]` | `Yes/No` |

Users can hold multiple roles; effective access is the union of allowed permissions, subject to applicable data scope. Avoid roles named after individual people and avoid a broad super-role when a narrower operational role is possible.

## 4. Direct grants and exceptional access

| Need | Allowed? | Controls |
| --- | --- | --- |
| Normal access | `No direct grant by default` | Assign a governed role. |
| Temporary emergency/special case | `Allowed only when approved` | Permission, reason, approver, start/end time, expiry, audit and review are mandatory. |
| Break-glass production access | `Only by operational policy` | Named identity, least privilege, time-bound access and post-use review. |

## 5. Enforcement and test matrix

| Scenario | Expected backend result | Test evidence |
| --- | --- | --- |
| Authorized role accesses permitted action | `[success/data scope]` | `[test]` |
| User with multiple roles | `[union of allowed permissions]` | `[test]` |
| Missing permission/direct URL/API call | `Denied; no protected data exposed` | `[test]` |
| Expired direct grant | `Denied and audited` | `[test]` |
| Different tenant context | `Denied/no data leakage` | `[test]` |

## 6. Approval

| Product owner | Security/technical owner | Date | Exceptions and expiry |
| --- | --- | --- | --- |
| `[name]` | `[name]` | `[date]` | `[link]` |
