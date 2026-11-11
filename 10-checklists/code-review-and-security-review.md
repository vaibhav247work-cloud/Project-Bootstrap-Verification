# Pull Request / Code Review and Security Review Checklist

## Pull request / code review

- [ ] Change is small, understandable and linked to requirement/decision; no unrelated refactor hides inside it.
- [ ] No backend project file exceeds the 800-line limit; split by responsibility when needed.
- [ ] Module-first placement, DTO boundaries, validation, global error contract and transaction behaviour follow standards.
- [ ] Authorization is enforced server-side; frontend permission checks are UX only.
- [ ] Trusted tenant context is used; no client value can select another tenant datasource or data scope.
- [ ] Queries are bounded/paginated and indexes/migrations match real access patterns.
- [ ] Secrets, tokens, PII and unsafe debug logs are absent; errors are safe.
- [ ] Tests cover the changed behaviour, failure path and permission/tenant boundary where relevant.
- [ ] API/schema/migration compatibility, idempotency and rollout impact are reviewed.

## Security review

- [ ] Threat model is current for auth, sensitive data, integration or boundary changes.
- [ ] Input/output, file/upload/export and dependency/webhook paths are validated and protected.
- [ ] Roles/direct grants follow expiry, approval and audit rules.
- [ ] Session/token, Redis cache keys and logging do not leak tenant or sensitive data.
- [ ] Secrets come from approved configuration; production access is least-privilege and auditable.
- [ ] Open findings have business/security approval, compensating control, owner and expiry.
