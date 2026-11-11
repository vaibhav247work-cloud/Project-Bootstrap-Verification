# Spring Boot: Database-per-Tenant with JPA/Hibernate and HikariCP

## Chosen model

One Spring Boot deployment can serve multiple isolated tenant databases:

```text
request for xyzschool
  → trusted tenant resolution
  → Hikari pool for tenant_xyzschool
  → Hibernate/JPA query only against tenant_xyzschool

request for jhkschool
  → trusted tenant resolution
  → Hikari pool for tenant_jhkschool
  → Hibernate/JPA query only against tenant_jhkschool
```

Tenant databases do not use cross-database business foreign keys or joins. Each has the same migration version/schema contract but independent data, configuration, users, roles, sessions, and audit records.

## Tenant resolution

1. **MUST** resolve tenant before JPA/Hibernate opens a tenant connection.
2. **MUST** derive tenant from a trusted server-controlled source: verified JWT/session tenant claim, mapped hostname/subdomain, or a server-side login/selection flow.
3. **MUST NOT** trust a raw `X-Tenant-ID`, request parameter, URL value, or frontend-selected database name without server-side membership/hostname validation.
4. **MUST** clear `TenantContext` in a `finally` block after every request and propagate it explicitly into async jobs; thread-local leakage is a tenant-data breach.
5. **MUST** reject a missing, unknown, inactive, or unauthorized tenant before any business database access.

## Hibernate/JPA implementation pattern

Use Hibernate database multitenancy:

```text
TenantContextFilter / authentication
  → TenantContext
  → CurrentTenantIdentifierResolver
  → MultiTenantConnectionProvider
  → tenant-specific HikariDataSource
  → EntityManager / Spring Data JPA
```

Implement and test:

- `CurrentTenantIdentifierResolver`: returns the already validated current tenant code.
- `MultiTenantConnectionProvider`: obtains/releases a JDBC connection from the correct tenant pool.
- `TenantDataSourceRegistry`: loads trusted tenant DB configuration and creates/evicts tenant Hikari pools.

Native SQL must use the same tenant-provided connection. Never bypass the multi-tenant provider with an arbitrary direct datasource/JDBC connection.

## Hikari pool policy

1. **MUST** create pools lazily on the first valid tenant request, not eagerly for every registered tenant.
2. **MUST** set `minimumIdle=0` (or another measured low value) for dynamic tenant pools so inactive tenants do not retain connections.
3. **MUST** return every borrowed JDBC connection to its Hikari pool at transaction/request completion. Application code never holds a connection beyond its transaction.
4. **MUST** cap total tenant pools per instance and close least-recently-used idle pools safely; closing a pool happens only when no active connection is borrowed.
5. **MUST** size `maximumPoolSize` from database connection capacity, application instance count, expected concurrent DB work, and tenant traffic—not a copied internet value.
6. **MUST** configure connection timeout, validation/keepalive strategy, max lifetime, idle timeout, and leak detection according to the managed DB/network behaviour; monitor pool saturation and acquisition time.
7. **MUST** store tenant datasource credentials in a secrets manager/config service, not the client request, source code, or ordinary tenant configuration table.
8. **MUST** expose health/metrics by tenant only with privacy-safe labels/cardinality; do not put uncontrolled tenant IDs into high-cardinality metrics.

### Two-level idle release

```text
Request/transaction completes
  → borrowed JDBC connection returns immediately to its Hikari pool

Tenant has no traffic
  → Hikari `idleTimeout` closes idle physical connections

Tenant pool unused past registry eviction period
  → TenantDataSourceRegistry removes and closes the whole HikariDataSource
```

The registry tracks last successful tenant access and active borrowed connection count. Its eviction callback closes a pool only if active count is zero; otherwise it postpones eviction. A bounded cache/registry (for example, maximum pool count plus expire-after-access) prevents unlimited tenant pools in one application instance.

Example external configuration shape (values are environment-specific):

```yaml
app:
  tenancy:
    datasource:
      default-pool:
        maximum-pool-size: 10
        minimum-idle: 0
        connection-timeout-ms: 3000
        idle-timeout-ms: 300000
```

The tenant registry maps a validated tenant code to its JDBC URL/credentials. Whether that registry lives in secure deployment configuration, a control database, or a managed tenant service is an ADR; it is not a business table inside a tenant database.

## Migrations and operations

1. **MUST** apply the same reviewed migration sequence to every active tenant DB and record tenant/schema version.
2. **MUST** support retry/resume when migration succeeds for some tenants but fails for another.
3. **MUST** back up, restore-test, monitor, and rotate credentials per tenant DB.
4. **MUST** integration-test tenant A/B isolation and verify a request cannot receive a connection from the wrong pool.
