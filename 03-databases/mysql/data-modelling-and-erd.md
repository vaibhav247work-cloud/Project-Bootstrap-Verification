# MySQL Data Modelling and ERD Rules

## Required design artifacts

Before a module's first migration, create a compact ERD and document:

- entities and their business meaning;
- primary keys and relationships;
- owner/tenant of each entity;
- lifecycle/status and deletion behaviour;
- business uniqueness rules;
- sensitive-data classification;
- expected query paths and data volume.

## Modelling rules

1. **MUST** model one real business concept per table. Do not create a generic catch-all table for unrelated concepts.
2. **MUST** normalize transactional data by default: each fact has one authoritative storage location.
3. **MUST** use a junction table for many-to-many relations and constrain duplicate pairs.
4. **MUST** keep derived values reproducible from source data unless materializing them has a documented performance, historical, or accounting need.
5. **MUST** record a historical snapshot when a past transaction must preserve facts that may later change, such as an order-line price, customer address used for delivery, or approval decision.
6. **MUST NOT** use an entity-attribute-value (EAV) table as the default data model. Use it only for a genuinely user-configurable schema with documented validation/query trade-offs.
7. **MAY** denormalize or create read/reporting tables only with an ADR covering source of truth, refresh method, staleness tolerance, and repair/rebuild plan.

## Table categories

| Category | Examples | Rules |
|---|---|---|
| Master/reference | countries, department types, status definitions | Stable codes, controlled changes, seed/administration policy |
| Transactional | bookings, invoices, attendance records | Full integrity, actor/timestamp fields, lifecycle and audit decision |
| Junction | user_roles, student_courses | FKs and unique relationship constraint |
| Audit/event | audit_logs, login_events | Append-only and retention controlled |
| Reporting/read model | daily summaries, dashboards | Not the source of truth; documented rebuild strategy |

## Reference and configuration data

1. **MUST** identify whether a value is code-managed, client-admin-managed, or project seed data.
2. **MUST** use stable business codes in addition to display names where integrations, permissions, or business logic depend on a value.
3. **MUST NOT** hard-delete a referenced value that is used by historical records; deactivate/archive it instead.
4. **SHOULD** use a project migration/seed process for baseline reference data so every environment is reproducible.
