# Reusable Product Modules

This directory defines the architecture, data schemas, and integration boundaries for cross-cutting business modules common to SaaS, ERP, workflow, and booking systems.

Rather than reinventing these foundations on each project, teams adapt these pre-designed module blueprints.

## Module Index

| Module | Purpose | File |
|---|---|---|
| **Notifications** | Multi-channel communication (Email, SMS, Push, In-App), template engine, provider adapters, opt-out preferences, delivery logs | [`notifications-module.md`](notifications-module.md) |
| **File Storage & Documents** | Object storage abstraction (S3/GCS/MinIO), tenant directory isolation, presigned URLs, MIME validation, virus scanning, retention lifecycle | [`file-storage-and-documents.md`](file-storage-and-documents.md) |
| **Audit Trail & Compliance** | Immutable append-only audit event logging, actor attribution, tamper-evident design, tenant filtering, compliance exports | [`audit-trail-service.md`](audit-trail-service.md) |
| **Bulk Import & Export** | Memory-safe streaming (preventing JVM OOM), chunked database ingestion, background worker execution, progress polling, download links | [`bulk-import-export.md`](bulk-import-export.md) |
| **Workflow & State Engine** | Finite State Machine (FSM) pattern, lifecycle transitions, transition guards, optimistic locking, audit on change | [`workflow-state-engine.md`](workflow-state-engine.md) |
