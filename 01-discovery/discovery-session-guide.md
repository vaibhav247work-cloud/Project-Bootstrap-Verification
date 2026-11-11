# Discovery Session Guide

Run discovery before any architecture or coding decision. The goal is to replace assumptions with documented, owner-confirmed answers. A skipped discovery item is a deferred risk, not a saved day.

## Who should attend

| Role | Responsibility in discovery |
|---|---|
| Product / business owner | Defines the problem, success, priority and budget constraints |
| Technical owner | Identifies feasibility, integration, security and operational constraints |
| End-user representative | Validates workflows, language and pain points |
| Security / compliance owner | Identifies data classification, regulatory and access constraints |
| Support / operations contact | Identifies support model, on-call expectations and known operational pain |

For a small product, one person may hold multiple roles. Record who was present and who was absent for each session.

## Session structure

Run in order. Do not start architecture or implementation until every MUST item has an answer or a named owner and due date.

### 1. Problem and outcome (30 min)

| Question | MUST answer |
|---|---|
| What problem exists today for which users? | Yes |
| What evidence proves it is real and worth solving? | Yes |
| What is the measurable success outcome? | Yes |
| What is explicitly out of scope? | Yes |
| What are the hard constraints (budget, date, regulation, existing system)? | Yes |

Capture: problem statement, success metrics with baseline and target, scope boundary, and hard constraints. These become Section 2 of the PRD template.

### 2. Users and access (30 min)

| Question | MUST answer |
|---|---|
| Who are the actors? List every human role and system identity. | Yes |
| What does each actor need to do, and what must they never do? | Yes |
| How does each actor log in? (email/password, SSO, passkey, service credential) | Yes |
| Who creates, activates, suspends, and deletes accounts? | Yes |
| Is there a tenant / organization model? How is isolation defined? | Yes |
| Are there regulatory or client contractual access requirements? | Yes |

Capture: actor/persona table, authentication methods, account lifecycle owners, and tenant isolation model. Feed into the security-and-access-matrix and PRD Section 3.

### 3. Critical workflows (45 min)

Walk through each important workflow end-to-end. For every workflow, capture:

- trigger (user action, schedule, integration event);
- happy path — step by step, actor by actor;
- failure paths (validation errors, permission denial, dependency failure, duplicate submission);
- data created, changed, or deleted;
- notifications, integrations, or side effects; and
- acceptance criteria — what observable result proves the workflow is correct.

Aim for 3–7 critical workflows. Use the workflow template in `09-templates/workflow-template.md`.

### 4. Data and integrations (30 min)

| Question | MUST answer |
|---|---|
| What data domains exist? What is their classification? | Yes |
| What is the retention and deletion rule for each domain? | Yes |
| What external systems are involved? What data crosses each boundary? | Yes |
| Are there imports, exports, bulk operations, or file uploads? | Yes |
| Are there reporting, search, or audit-access requirements? | Yes |

Capture: data domain table with classification and retention. Feed into PRD Section 6 and the data-model template.

### 5. Non-functional requirements (20 min)

| Area | Question | MUST answer |
|---|---|---|
| Scale | How many users, tenants, records, and requests at peak? | Yes |
| Availability | What are the support hours and acceptable downtime? | Yes |
| Performance | What is the acceptable response time for critical journeys? | Yes |
| Devices / browsers | What must the product support? | Yes |
| Accessibility | What conformance level is required? | Yes |
| Localization | What languages, timezones, and currencies are needed? | Yes |
| Compliance | GDPR, HIPAA, ISO 27001, sector regulation, client contract? | Yes |

### 6. Operational readiness (15 min)

| Question | MUST answer |
|---|---|
| Who owns production support and what are the response hours? | Yes |
| Who is the named escalation contact for security and data incidents? | Yes |
| What is the backup and disaster-recovery expectation? | Yes |
| Are there existing monitoring, deployment, or infrastructure constraints? | Yes |

## Outputs and next steps

At the end of discovery, every item below must have an answer or a named owner + due date. An item with no owner is an unaccepted risk.

- [ ] `01-discovery/project-brief.md` completed and reviewed.
- [ ] `06-security/security-and-access-matrix.md` Steps 1–5 completed.
- [ ] All critical workflows documented (use `09-templates/workflow-template.md`).
- [ ] Data classification and retention decisions recorded.
- [ ] Non-functional targets agreed and written into the PRD.
- [ ] Operational and support model agreed.
- [ ] Open decisions have named owner and due date — not a blank.
- [ ] PRD approved before architecture or implementation starts (`09-templates/product-requirements-document-template.md`).

## Tips

- Record verbatim where a stakeholder is uncertain — do not clean it into false confidence.
- A workflow described in five minutes is probably under-specified. Ask: "What happens when it fails?" and "Who is not allowed to do this?"
- If two stakeholders give contradictory answers, capture both and escalate before design starts.
- Discovery is not a one-meeting event. Schedule follow-ups for missing answers with named deadlines.
