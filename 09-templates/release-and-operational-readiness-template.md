# Release, Rollout, Rollback and Operational Readiness Template

> Complete before production approval. A release is not ready merely because it builds successfully.

| Field | Value |
| --- | --- |
| Release/version | `[immutable artifact/version]` |
| Scope / PRD/stories | `[links]` |
| Release owner | `[role]` |
| Change window | `[date/time/timezone]` |
| Tenant sequencing | `[pilot/waves/all; approved plan]` |

## Go/no-go evidence

| Gate | Result/evidence | Owner |
| --- | --- | --- |
| Approved change scope and quality evidence | `[link]` | `[role]` |
| Database migration/backfill tested and compatible | `[link]` | `[role]` |
| Security/privacy and access review complete | `[link]` | `[role]` |
| Monitoring, alerts, dashboard and runbook ready | `[link]` | `[role]` |
| Backup/recovery implications reviewed | `[link]` | `[role]` |
| Client/support communication and contacts ready | `[link]` | `[role]` |

## Execution, rollback and verification

| Stage | Action | Owner | Success signal | Stop/rollback trigger | Evidence |
| --- | --- | --- | --- | --- | --- |
| Pre-deploy | `[action]` | `[role]` | `[signal]` | `[trigger]` | `[link]` |
| Deploy/wave | `[action]` | `[role]` | `[signal]` | `[trigger]` | `[link]` |
| Verify | `[critical journeys/SLO]` | `[role]` | `[signal]` | `[trigger]` | `[link]` |

Rollback must state artifact/config/feature-flag/database forward-fix method; never assume a destructive migration can simply be reversed.
