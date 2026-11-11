# Threat Model Template

> Complete for every new product, externally exposed integration, authentication/authorization change, sensitive-data workflow, major tenant/data boundary change, or material architecture change. Focus on realistic abuse and failure paths; link the solution architecture and security decisions.

## 1. Scope

| Field | Value |
| --- | --- |
| Product / feature | `[name]` |
| PRD / architecture / ADR | `[links]` |
| Reviewer(s) | `[roles/names]` |
| Date / status | `[date; draft/reviewed/accepted]` |
| Data classification | `[public/internal/personal/sensitive]` |
| Tenant boundary | `Each tenant has an isolated database; state any approved exception.` |

## 2. Assets, actors and trust boundaries

| Asset / boundary | Why it matters | Actors with legitimate access | Protection required |
| --- | --- | --- | --- |
| `[user identity/session]` | `[impact if compromised]` | `[roles/systems]` | `[controls]` |
| `[tenant data]` | `[impact]` | `[roles]` | `[controls]` |
| `[integration/secret]` | `[impact]` | `[service]` | `[controls]` |

List entry points and trust boundaries: browser/mobile client, API gateway, Spring Boot modules, tenant datasource selection, MySQL tenant DB, Redis, background jobs, CI/CD, administrators and third-party APIs. Link a data-flow diagram; label data direction and authentication on every crossing.

## 3. Abuse and failure cases

| ID | Threat / abuse case | Entry point / precondition | Impact | Existing controls | Residual risk | Mitigation / owner / due date | Verification |
| --- | --- | --- | --- | --- | --- | --- | --- |
| TM-01 | `[e.g., forged tenant identifier]` | `[path]` | `[impact]` | `[controls]` | `Low/Med/High` | `[action]` | `[test/review]` |

Consider at least:

- authentication bypass, credential stuffing, session/token theft and unsafe recovery;
- broken authorization, privilege escalation, direct emergency-grant misuse and frontend-only enforcement;
- tenant-context spoofing, cross-tenant reads/writes and unsafe administrator tooling;
- injection, unsafe file upload, insecure deserialization and unvalidated dynamic configuration;
- sensitive-data exposure through API, logs, exports, browser storage, cache, backups or error messages;
- rate abuse, denial of service, unbounded queries/uploads and resource exhaustion;
- integration/webhook spoofing, replay, timeout/retry duplication and compromised dependency credentials;
- supply-chain, CI/CD, secret exposure and unsafe production access; and
- data loss/corruption, unsafe migration/restore and audit-evidence loss.

## 4. Required security decisions

| Area | Decision | Evidence / link |
| --- | --- | --- |
| Authentication/session | `[email, SSO/passkey requirement, access/refresh rules]` | `[link]` |
| Authorization | `[RBAC/direct-grant expiry/audit rule]` | `[link]` |
| Tenant resolution | `[trusted server-side source; no client-selected DB]` | `[link]` |
| Input/output protection | `[validation, encoding, upload policy]` | `[link]` |
| Encryption/secrets | `[TLS, secret manager, key handling]` | `[link]` |
| Logging/audit | `[safe telemetry and required events]` | `[link]` |
| Availability/resilience | `[limits, timeout, recovery]` | `[link]` |

## 5. Sign-off and re-review triggers

| Decision | Owner | Date | Notes |
| --- | --- | --- | --- |
| `Accepted / changes required / risk accepted temporarily` | `[role/name]` | `[date]` | `[expiry and reason if exception]` |

Re-review after a material auth, data-classification, tenant-boundary, integration, deployment or architecture change; after an incident; and at the product-defined periodic review.
