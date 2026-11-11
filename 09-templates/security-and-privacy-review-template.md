# Security and Privacy Review Template

> Use before production release for features that process personal/sensitive data, change login/access, expose an API/integration, add export/import/upload, or have a material threat-model finding. This is a review record, not a substitute for testing.

## Review metadata

| Field | Value |
| --- | --- |
| Feature / release | `[name]` |
| PRD / architecture / threat model | `[links]` |
| Data classification | `[answer]` |
| Reviewers | `[roles/names]` |
| Outcome | `Approved / approved with time-bound exception / changes required` |

## Review checklist

| Area | Question | Result / evidence | Owner/action if incomplete |
| --- | --- | --- | --- |
| Data minimization | Is every collected/stored/exported field necessary? | `[answer]` | `[action]` |
| Access | Are backend permission and tenant checks tested for every protected action? | `[answer]` | `[action]` |
| Authentication/session | Are login, logout, refresh, SSO/passkey and recovery needs safely configured? | `[answer]` | `[action]` |
| Input/output | Are validation, encoding, upload/download and error responses safe? | `[answer]` | `[action]` |
| Secrets | Are secrets absent from code, logs, frontend bundles and documents? | `[answer]` | `[action]` |
| Encryption | Is TLS enforced and is stored-data protection appropriate to classification? | `[answer]` | `[action]` |
| Audit/telemetry | Are security events recorded without leaking sensitive values? | `[answer]` | `[action]` |
| Retention/deletion | Are retention, deletion, backup and restore obligations known? | `[answer]` | `[action]` |
| Sharing/integrations | Is shared data minimized; auth, webhook validation and failure handling defined? | `[answer]` | `[action]` |
| Abuse/resilience | Are rate limits, timeouts, retries and resource limits suitable? | `[answer]` | `[action]` |
| Testing | Are threat-model mitigations and critical abuse paths tested? | `[answer]` | `[action]` |

## Exception record

No permanent undocumented exception is allowed.

| Exception | Risk | Business owner | Technical/security approver | Compensating control | Expiry/review date | Tracking link |
| --- | --- | --- | --- | --- | --- | --- |
| `[item]` | `[risk]` | `[name]` | `[name]` | `[control]` | `[date]` | `[link]` |
