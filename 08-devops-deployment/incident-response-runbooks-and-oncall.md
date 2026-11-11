# Incident Response, Runbooks, On-Call and Post-Incident Improvement Standard

## 1. Purpose and scope

Every production product needs a repeatable response for availability, performance, security, data-integrity, tenant-isolation, dependency and deployment incidents. Protect people and customer data, stabilize the service, communicate honestly, recover safely and learn without blame.

Client contract, legal, privacy and security obligations can impose stricter rules. The PRD and support agreement define actual support hours, response targets and client contacts; this playbook does not assume every product has 24x7 support.

## 2. Incident severity

| Level | Typical impact | Required response |
| --- | --- | --- |
| SEV1 | Data exposure, tenant-boundary failure, authentication bypass, irreversible data loss/corruption, or broad critical outage | Declare immediately; assign roles; contain harm; involve security/privacy and designated business/client contacts. |
| SEV2 | Major core workflow unavailable or seriously degraded for one or more tenants, without a reasonable workaround | Page the responsible on-call path promptly; stabilize, communicate and track recovery. |
| SEV3 | Limited tenant or feature impact with a safe workaround | Create a tracked incident or defect; respond in the agreed support window. |
| SEV4 | Low impact, cosmetic issue or improvement | Normal backlog handling. |

When uncertain, start at the higher severity and reduce it only when evidence supports doing so. Severity measures impact, not who made a mistake.

## 3. Incident roles

For SEV1 and SEV2, name these roles in the incident channel/log. In a small team one person may cover more than one role, but ownership remains explicit.

| Role | Responsibility |
| --- | --- |
| Incident commander | Coordinates response, prioritizes safety and recovery, decides when to pause rollout or escalate. |
| Technical lead | Investigates, proposes safe mitigations and verifies recovery. |
| Communications lead | Writes internal and approved external status updates. |
| Scribe | Maintains a timestamped factual timeline, decisions and evidence links. |
| Security/privacy owner | Leads security, privacy or tenant-isolation handling and required escalation. |
| Product/client decision owner | Makes approved business or client-impact decisions. |

The incident commander coordinates; they need not be the deepest technical expert. Emergency changes still follow the documented emergency-change process.

## 4. Response lifecycle

1. Detect and declare: acknowledge the alert/report, open the incident record, set initial severity and assign roles.
2. Stabilize and contain: pause harmful rollout/write paths when justified; use a known-safe rollback or feature flag; protect tenant boundaries and evidence.
3. Assess: identify affected tenants, workflows, versions, time range and data risk. Keep facts separate from assumptions.
4. Communicate: state impact, mitigation, known/unknown information and next update time.
5. Recover and verify: use the least-risk mitigation; validate service, data integrity, tenant isolation and monitoring before declaring recovery.
6. Close and improve: complete the timeline, post-incident review and tracked follow-up actions.

Never perform unrecorded production changes, delete evidence, or run destructive data repair before backup/snapshot and approval checks.

## 5. First-response checklist

- Acknowledge, declare severity and create the incident channel/log.
- Name incident commander and technical lead; add security/privacy owner for any data or access concern.
- Capture service, environment, tenant/database, version, alert, start time and observed impact.
- Check recent deployments, feature flags, dependencies, dashboards, logs and audit trails.
- Pause a rollout or disable a feature only through its documented safe control; record who made the change and why.
- Preserve relevant logs, traces, audit records and configuration evidence with responder-only access.
- State what is known, unknown, being tried and the next update time.

Treat suspected credential, privacy, data-exposure or tenant-boundary events as security incidents: restrict the smallest safe scope, preserve evidence, follow the security escalation path, and allow only authorized external/regulatory communication.

For data-integrity/database incidents, stop harmful writes if appropriate. Do not improvise direct production repairs: identify the tenant database, confirm backup/restore options, follow the recovery runbook and validate before reopening writes.

## 6. Runbook standard

Each production service and operational alert has an owned, versioned runbook accessible during an outage. It contains purpose, owner/escalation contacts, last test date, required least-privilege access, a short decision tree, exact target validation, safe actions with expected output, rollback/verification, relevant dashboard/alert links, escalation and stop conditions.

Runbooks never contain plaintext credentials, tokens or customer personal data. Use variables and approved secret retrieval. Test important runbooks through drills or non-production recovery exercises after material changes.

Minimum runbook catalogue:

- SLO/error-budget burn or widespread application failure;
- failed rollout, rollback and feature-flag disablement;
- login, session, JWT/refresh-token or Redis failure;
- tenant datasource, connection-pool or migration failure;
- MySQL outage, restore and point-in-time recovery;
- secret/credential compromise or rotation;
- security, privacy or tenant-boundary concern;
- background-job, outbox or queue backlog and safe replay;
- third-party provider failure;
- certificate, DNS or network failure; and
- capacity, traffic surge or denial-of-service response.

## 7. On-call and production access

Each supported product records support coverage, primary/backup responder, escalation contacts, acknowledgement and communication targets, plus client contact route. Select these during discovery and operational readiness, not by assumption.

On-call responders use named accounts, least privilege and approved just-in-time or break-glass access. Break-glass access is time-bound, audited and reviewed after use. Production access is never shared, permanent by default or used to bypass tenant isolation.

Handoffs include open incidents, degraded services, pending changes, known risks and ownership. Test alert routes, contacts and access paths periodically.

## 8. Communication

Maintain one timestamped incident timeline. Updates state impact, scope, current mitigation, known/unknown information, customer action if any and next update time. Never include secrets, personal data, speculation or blame.

Only designated communications or authorized client/security contacts issue external updates. Contractual, privacy, regulatory and legal notifications follow the applicable agreement and process.

## 9. Blameless post-incident review

Every SEV1/SEV2 and recurring SEV3 receives a proportional review on the product-defined cadence. The review examines systems, decisions, safeguards and conditions, not personal fault.

Record impact/scope, detection-to-recovery timeline, technical cause and contributing conditions (mark uncertainty), what worked, recovery friction, and corrective/preventive actions. Every action gets owner, due date and verification method. Track actions to closure and add tests, alerts, dashboards, infrastructure controls or runbook updates as needed.

## 10. Exercises, measures and production readiness

Run proportionate tabletop/recovery exercises for rollback, alert routing, restore, tenant-boundary containment, secret rotation, dependency failure and capacity degradation. Use service indicators with incident frequency, repeat incidents, time to acknowledge/recover, alert noise, runbook coverage and restore-test results to improve systems and staffing, not rank individuals.

Before a new production service or critical workflow is released, confirm its owner, support/escalation path, health/SLO signals, tested rollback, relevant runbooks, secure access path, backup/recovery plan and incident communication route. A feature without an operable failure path is not production-ready.
