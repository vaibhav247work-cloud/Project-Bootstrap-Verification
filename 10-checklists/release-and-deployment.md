# Release Readiness and Production Deployment / Rollback Checklist

## Release readiness

- [ ] Immutable artifact/version, scope and owner are identified.
- [ ] Required automated tests, contract/integration tests and UAT evidence pass or have approved exception.
- [ ] Database migration/backfill is backward-compatible, tested and sequenced per tenant.
- [ ] Security/privacy review, known risks and data/retention implications are resolved or approved.
- [ ] Dashboards, alerts, SLO signals and relevant runbooks exist and have owners.
- [ ] Rollout wave/feature-flag plan, rollback/forward-fix decision and stop triggers are documented.
- [ ] Backup/recovery, client/support communication and on-call coverage are ready.

## Deployment and rollback

- [ ] Confirm environment, artifact digest/version, tenant/wave and change window before action.
- [ ] Record deploy time, operator, configuration/flag state and migration version.
- [ ] Observe technical and business success signals after each wave.
- [ ] Stop rollout on documented trigger; declare incident if customer/data/security impact needs it.
- [ ] Roll back artifact/config/flag through approved path; do not make unrecorded production repairs.
- [ ] Validate critical flows, tenant isolation, error rate, jobs/outbox and data consistency after deploy or rollback.
