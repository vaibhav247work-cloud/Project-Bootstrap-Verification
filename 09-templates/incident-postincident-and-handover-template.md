# Incident, Post-Incident and Handover Templates

## Incident record

| Field | Value |
| --- | --- |
| ID/severity/start time | `[answer]` |
| Affected service/tenant/workflow | `[answer]` |
| Incident commander / technical lead / communications | `[roles]` |
| Customer/data/security impact | `[facts only]` |
| Status channel/timeline | `[link]` |

| Time | Observation/decision/action | Owner | Evidence/link |
| --- | --- | --- | --- |
| `[time]` | `[fact]` | `[role]` | `[link]` |

## Post-incident review

- Impact and scope: `[answer]`
- Detection-to-recovery timeline: `[link]`
- Technical cause and contributing conditions: `[answer; mark uncertainty]`
- What worked / what made recovery harder: `[answer]`

| Follow-up action | Owner | Due date | Verification | Status |
| --- | --- | --- | --- | --- |
| `[action]` | `[role]` | `[date]` | `[proof]` | `[open/done]` |

## Project/service handover

| Area | Required handover item | Location/owner |
| --- | --- | --- |
| Product | PRD, workflows, decisions, support contacts | `[link]` |
| Code/API/data | repositories, OpenAPI, migrations, tenant rules | `[link]` |
| Security | permissions, secrets ownership, threat model, exceptions | `[link]` |
| Operations | dashboards, alerts, runbooks, backup/restore proof, on-call | `[link]` |
| Quality/release | test evidence, known defects/risks, rollback history | `[link]` |

Handover is accepted only when the receiving owner can locate, access and operate the service without relying on undocumented personal knowledge.
