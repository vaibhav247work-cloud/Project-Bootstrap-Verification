# Authorization Enforcement Standard

## Every protected command checks four layers

```text
authenticated active session
→ required permission
→ tenant and data/relationship scope
→ current business state
```

1. **MUST** enforce authorization in application/use-case services, not only controller annotations or frontend visibility.
2. **MUST** use permission constants from the permission catalog; no scattered string literals or role-name checks.
3. **MUST** validate tenant context and record ownership/relationship before reading, changing, exporting, attaching or deleting a record.
4. **MUST** use explicit policy/evaluator classes for non-trivial scope rules, e.g. `BookingAuthorizationPolicy` or `TeacherClassScopePolicy`.
5. **MUST** re-check business state for commands, e.g. only a draft booking can be submitted and only an approved invoice can be paid.
6. **MUST** deny by default and return safe 403/404 behaviour without confirming cross-tenant record existence.
7. **MUST** test allow and deny paths for every sensitive action.

Spring Security annotations may provide coarse endpoint protection; they do not replace object-level authorization.
