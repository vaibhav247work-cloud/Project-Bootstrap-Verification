# Security and Access Matrix

Complete this before implementing any protected feature. The backend is the authority for every access decision; hiding a UI element is never a substitute for server-side enforcement.

## Step 1 — Data classification

Classify every data domain the product handles. This drives encryption, retention, access-log, and deletion requirements.

| Data domain | Classification | Examples | Stored encrypted at rest | Logged access | Retention / deletion rule |
|---|---|---|---|---|---|
| Public | Public | Product catalogue, public pages | No | No | Keep while needed |
| Internal | Internal | Bookings, schedules, reports | Recommended | On export | Per business policy |
| Personal | Personal | Name, email, mobile, address | Yes | Yes — access and export | Legal/privacy minimum |
| Sensitive | Sensitive | Passwords (hash only), health, payment, documents, audit logs | Yes | Yes — every read/change | Regulatory / contractual |

Replace the example rows with actual domains for your product.

## Step 2 — Environment control baseline

| Control | Local / CI | Dev / SIT | UAT | Production |
|---|---|---|---|---|
| HTTPS enforced | No (local) / Yes (CI) | Yes | Yes | Yes — HSTS required |
| Real personal/sensitive data permitted | No | No — synthetic only | No — masked / synthetic only | Yes — live data |
| Production secrets accessible | No | No | No | Yes — approved secret manager only |
| Access logging / audit trail | Optional | Yes | Yes | Yes — mandatory |
| Backup / PITR enabled | No | No | Optional | Yes — mandatory |
| External email / SMS / payment | Mocked | Sandbox | Sandbox | Live — approved credentials |
| Public internet exposure | No | Restricted | Restricted | Defined ingress only |

## Step 3 — Resource and permission matrix

Define every protected resource, action, and allowed roles before implementation. Use the permission naming format `<module>.<resource>.<action>`.

| Permission | Resource / action | Allowed roles | Data scope | Requires MFA / re-auth | Audit required | Notes |
|---|---|---|---|---|---|---|
| `employee.profile.view` | View employee record | `hr_manager`, `org_admin` | Own organization | No | On bulk export | — |
| `employee.profile.update` | Edit employee record | `hr_manager` | Own organization | No | Yes | — |
| `role.assignment.manage` | Assign / revoke roles | `org_admin` | Own organization | Yes | Yes | Sensitive — elevated |
| `report.export` | Download report | `hr_manager`, `org_admin` | Own organization | No | Yes — with actor | Limit file size and rate |
| `platform.tenant.manage` | Create / suspend tenant | `platform_admin` | All tenants | Yes | Yes | Platform-level only |

Replace the example rows with actual permissions. Every row maps directly to a backend enforcement point.

## Step 4 — Mandatory decisions

Record these before authentication or protected features are implemented. Write `Decision needed — owner: [name], due: [date]` for any unknown.

| Decision | Answer |
|---|---|
| Primary login identifier | `Email` (default) — or ADR required |
| Account creation methods enabled | `[self-registration / invite / admin-created / SSO JIT / import]` |
| Session model | `Short-lived JWT + rotating server-tracked refresh token` (default) |
| Access token lifetime | `15 minutes` (default) — adjust with rationale |
| Refresh session idle / absolute expiry | `30 days idle / 90 days absolute` (default) |
| MFA — mandatory for | `Platform admins, tenant admins, and financial / security-config actions` (MUST) |
| MFA — available for | `All users` (SHOULD) |
| Tenant isolation model | `[one DB per tenant / shared schema / single-org deployment]` |
| Sensitive actions requiring re-authentication | `[role management, export, payment, account disable, break-glass access]` |
| Audit log retention | `[minimum 2 years for auth / financial / permission events — adjust per legal/client policy]` |
| Data export / download restrictions | `[rate limit, permission gate, actor audit, max file size]` |
| Secret manager / credential store | `[approved platform secret manager]` |
| Backup frequency and PITR requirement | `[daily full + binary log for PITR — define RPO]` |

## Step 5 — Access review commitment

| Question | Answer / owner |
|---|---|
| What access does this user effectively have, and why? | Answered by: `[role/tool]` |
| Which users hold this sensitive permission? | Answered by: `[role/tool]` |
| Privileged-role review cadence | `[quarterly / per-project risk]` |
| Dormant account review cadence | `[quarterly]` |
