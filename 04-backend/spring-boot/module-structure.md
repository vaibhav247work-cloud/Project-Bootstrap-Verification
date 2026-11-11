# Spring Boot Module-First Structure

## Rule

Organize by business capability first, not by global technical layer. A booking change should usually stay inside `booking/`; a loyalty change should stay inside `loyalty/`.

```text
com.company.product/
├─ bootstrap/                         # Application entry point, startup wiring
├─ platform/                          # Technical cross-cutting infrastructure
│  ├─ config/                         # typed configuration, feature wiring
│  ├─ database/                       # JPA/Hibernate, migrations, tenant datasource routing
│  ├─ security/                       # Spring Security, JWT/session plumbing
│  ├─ web/                            # Problem Details, request IDs, CORS, filters
│  ├─ cache/                          # Redis clients, cache policies
│  ├─ integration/                    # shared HTTP client factory/resilience primitives
│  └─ observability/                  # metrics, tracing, structured logs
├─ shared/                            # Small stable technical/value primitives only
│  ├─ error/                          # shared exception/error contracts
│  ├─ validation/                     # reusable non-business validators
│  └─ types/                          # money, IDs, paging contracts only when truly universal
└─ modules/
   ├─ booking/
   │  ├─ api/                         # BookingController, request/response DTOs
   │  ├─ application/                 # use cases, commands, transaction boundary
   │  ├─ domain/                      # Booking entity, rules, value objects, repository interface
   │  ├─ infrastructure/              # JPA repository adapter, third-party adapter
   │  └─ BookingPermissionCatalog.java
   ├─ loyalty/
   │  ├─ api/
   │  ├─ application/
   │  ├─ domain/
   │  ├─ infrastructure/
   │  └─ LoyaltyPermissionCatalog.java
   └─ identity_access/
      ├─ api/
      ├─ application/
      ├─ domain/
      └─ infrastructure/
```

For a small module, do not create empty layers mechanically. Start with `api`, `application`, and `domain`; add `infrastructure` when persistence/integration needs an adapter boundary.

## Dependency direction

```text
api → application → domain
infrastructure → domain/application contracts
platform → technical support only
```

1. **MUST NOT** let one module access another module's JPA repository/entity directly.
2. **MUST** integrate modules through a narrow application service/interface, domain event, or explicit API contract.
3. **MUST** keep controllers thin: validation/DTO mapping only, then call a use case.
4. **MUST** keep transaction boundaries in the application/use-case layer, not controllers.
5. **MUST** keep JPA entities/infrastructure mappings inside their owning module.

## What goes in `shared` / `common`

`shared` is not a generic dumping folder. It may contain only stable, non-business code used by multiple modules: error format, pagination type, money value object, request ID, generic validation annotation.

It **must not** contain booking rules, employee states, loyalty calculations, module DTOs, random utilities, or a giant generic service/repository. If only one module uses it, it stays in that module. If two modules share a business concept, first decide whether it is a new module or an explicit contract—not an automatic move to `common`.

Avoid a root `model/`, `controller/`, `service/`, or `repository/` folder across the entire application.

See `../backend-code-quality.md` for source-file size, split, DTO/entity, and injection rules.

Related backend standards: `api-standard.md`, `input-validation.md`, `authorization-enforcement.md`, `transactions-events-outbox.md`, `background-jobs.md`, `cache-standard.md`, `observability.md`, `configuration-secrets.md`, `file-handling.md`, and `testing-and-ci.md`.
