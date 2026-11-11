# MySQL Playbook

Use these rules for the default relational database in this engineering system.

## Baseline

- Engine: InnoDB
- Character set: `utf8mb4`
- Collation: choose one explicitly for the product; use it consistently
- Time standard: store instants in UTC; convert only at the API/UI boundary
- Schema change method: versioned migrations only

## Required companion documents

- Data model / ERD for each module
- Migration plan for every schema change
- Index rationale for non-obvious or composite indexes
- Retention/deletion decision for each sensitive or high-volume entity

Read `schema-rules.md`, `data-modelling-and-erd.md`, `indexing-and-queries.md`, `concurrency-and-consistency.md`, `reference-and-configuration-data.md`, `json-and-scale-ready-design.md`, `performance-and-scale.md`, `database-testing.md`, and `migrations-and-operations.md` before adding a production table. For authentication/authorization tables, also read `identity-access-schema.md`. For multi-organization products, read `multi-tenancy.md`; for sensitive or auditable data, read `audit-log-standard.md` and `sensitive-data-and-privacy.md`.
