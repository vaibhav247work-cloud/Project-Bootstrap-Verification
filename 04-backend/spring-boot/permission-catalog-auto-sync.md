# Spring Boot Pattern: Permission Catalog Auto-Sync

## Scope and status

This document completes the third part of the Dynamic Permission Catalog work.

| Sub-step | Status |
|---|---|
| Permission naming format | Done: `06-reusable-modules/identity-access/dynamic-permissions.md` |
| MySQL structure | Done: `03-databases/mysql/identity-access-schema.md` |
| Spring Boot catalog auto-sync | Done: this document |
| Full authentication/JWT/SSO implementation standard | Pending: next phase |

## Design

Permissions are declared in Java code, not typed manually in an admin database screen.

```text
Java PermissionDefinition records
        ↓
PermissionCatalogProvider(s), one per backend module
        ↓
PermissionCatalogSyncService
        ↓
permissions table
```

Use one shared definition object and constants to prevent spelling mismatches between API protection, sync, and tests.

```java
public record PermissionDefinition(
    String code,
    String name,
    String module,
    ScopeLevel scopeLevel,
    RiskLevel riskLevel,
    String description
) {}
```

```java
public interface PermissionCatalogProvider {
    Collection<PermissionDefinition> permissions();
}
```

Example module provider:

```java
@Component
final class EmployeePermissionCatalog implements PermissionCatalogProvider {
    static final String VIEW = "employee.profile.view";
    static final String EXPORT = "employee.profile.export";

    public Collection<PermissionDefinition> permissions() {
        return List.of(
            new PermissionDefinition(VIEW, "View employee profile", "employees", ScopeLevel.ORGANIZATION, RiskLevel.NORMAL, "..."),
            new PermissionDefinition(EXPORT, "Export employee profiles", "employees", ScopeLevel.ORGANIZATION, RiskLevel.HIGH, "...")
        );
    }
}
```

The exact Java packaging can vary, but every code reference must use the same catalog definition/constant. No raw string permission code may be repeated across controllers and services.

## Sync algorithm

1. Collect all definitions from every `PermissionCatalogProvider`.
2. Validate code format and uniqueness before writing anything.
3. In one transaction, load existing catalog records by code.
4. Insert definitions absent from the database.
5. Update only safe metadata for existing definitions: name, description, module, and risk classification.
6. Do **not** delete database permissions merely because a declaration disappeared.
7. Mark removal/deprecation only through an explicit reviewed migration/ADR.
8. Emit a structured audit/deployment event with created and updated codes.

The process must be idempotent: rerunning the same release produces no duplicate records or changed access grants.

## When it runs

| Environment | Recommended execution |
|---|---|
| Local development | Automatically on application startup |
| Dev/staging | Automatically once per deployment/job |
| Production | A single controlled deployment/migration job before app instances receive traffic |

**Do not run catalog writes concurrently in every production application instance.** Use a one-time command/profile such as `--app.permission-catalog.sync=true`, or a dedicated deployment job.

The production sync job uses the migration/catalog-write database account. Normal API runtime accounts do not receive catalog-administration privileges.

## Failure behaviour

1. **MUST** fail the deployment/job if duplicate or invalid declarations exist.
2. **MUST** fail safely if a protected permission is absent after sync.
3. **MUST NOT** continue with an unknown permission as implicitly allowed.
4. **SHOULD** log the code and deployment version, never secrets or user tokens.

## Integration with authorization

This sync process registers permissions; it does not grant them to custom roles.

```text
new catalog permission
  -> platform_admin / organization_admin / module_admin: dynamically effective by policy
  -> custom_role: denied until explicit role-permission assignment
  -> direct user override: only the existing approved, temporary exception path
```

Spring Security enforcement will be documented in the next authentication/authorization phase. The requirement for that phase is: backend checks use catalog constants and must validate permission, organization, scope/relationship, and business state.

## Required tests

- Catalog code is unique and matches the naming format.
- Sync inserts a missing permission.
- Rerunning sync does not create duplicates.
- Sync updates safe metadata only.
- Missing/invalid declarations fail the sync.
- Every protected endpoint references a known catalog permission.
- A custom role does not receive a newly synced permission by default.
