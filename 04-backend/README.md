# Backend Engineering Standards

Default backend is **Java + Spring Boot**, structured as a modular monolith. Other frameworks or distributed event architectures are allowed when a documented requirement justifies them.

## Directory Map

```text
04-backend/
├─ backend-code-quality.md                      # Source file limits (800 lines), injection, mapping rules
├─ messaging-and-event-broker-standard.md       # Kafka / RabbitMQ broker standards, DLQ, partitioning
└─ spring-boot/
   ├─ module-structure.md                       # Business-capability-first package organization
   ├─ architecture-testing-archunit.md          # Automated ArchUnit tests enforcing boundaries & imports
   ├─ api-standard.md                           # REST, OpenAPI contracts, pagination, error schemas
   ├─ authentication-implementation.md          # JWT, rotating refresh token filters
   ├─ authorization-enforcement.md              # Server-side permission & scope checks
   ├─ background-jobs.md                        # Scheduled tasks & background workers
   ├─ cache-standard.md                         # Redis caching & invalidation rules
   ├─ configuration-secrets.md                  # Externalized configuration properties
   ├─ database-per-tenant-jpa.md                # Hikari dynamic multi-tenant routing
   ├─ file-handling.md                          # Upload validation, storage abstraction, signed URLs
   ├─ global-error-handling.md                  # RFC 7807 Problem Details controller advice
   ├─ http-client-standard.md                   # Pooled third-party HTTP clients & resilience
   ├─ input-validation.md                       # Bean Validation & request constraints
   ├─ observability.md                          # Structured logs, correlation IDs, OpenTelemetry
   ├─ permission-catalog-auto-sync.md           # Dynamic catalog registration
   ├─ testing-and-ci.md                         # JUnit 5, Testcontainers, quality gates
   ├─ transactions-events-outbox.md             # Outbox pattern for atomic event publication
   └─ webhook-ingestion-standard.md             # HMAC signature verification, replay protection & async ingest
```
