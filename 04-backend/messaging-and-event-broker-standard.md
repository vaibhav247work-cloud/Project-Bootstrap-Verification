# Messaging and Event Broker Standard

## Purpose

This standard governs asynchronous event-driven architecture when workloads outgrow the local transactional database outbox pattern, or when multiple services/systems must consume domain events asynchronously at scale.

For smaller products and modular monoliths, the **Transactional Outbox in MySQL** (`04-backend/spring-boot/transactions-events-outbox.md`) remains the default. Use an external event broker (e.g., Apache Kafka, RabbitMQ, AWS SQS/SNS) only when throughput, decoupling, or event retention requirements justify it.

---

## 1. Decision Guide: Database Outbox vs. Dedicated Event Broker

| Criteria | Database Outbox (Default) | Dedicated Event Broker (Kafka / RabbitMQ) |
|---|---|---|
| **Event Volume** | < 1,000 events/sec | > 1,000 events/sec or high-throughput telemetry |
| **Consumer Count** | Local background workers within same app | Multiple independent microservices or external systems |
| **Ordering Guarantee** | Guaranteed per database transaction | Guaranteed per topic partition / exchange key |
| **Operational Overhead** | Zero extra infrastructure (uses existing MySQL) | High (requires broker cluster management, monitoring, rebalancing) |

---

## 2. Producer Standard: Guaranteed Delivery

1. **MUST** use the **Transactional Outbox Pattern** to publish domain events. An application must never write to MySQL and an external event broker in separate, non-atomic steps.
   ```text
   Application Use Case
         ↓ (One Local DB Transaction)
   [Business Record Update] + [outbox_events Table Insert]
         ↓ (Committed)
   Outbox Publisher Process (Debezium / Spring Scheduled Worker)
         ↓
   External Event Broker (Kafka / RabbitMQ)
   ```
2. **MUST** specify a deterministic **Partition / Routing Key**:
   - In multi-tenant systems, the partition key is typically `tenant_id` or `tenant_id:aggregate_id`.
   - All events for a specific business entity (e.g., `booking-123`) must route to the same partition to guarantee strict in-order processing.
3. **MUST** attach mandatory event envelope metadata:
   ```json
   {
     "eventId": "01J...",
     "eventType": "booking.confirmed",
     "eventVersion": 1,
     "occurredAt": "2026-11-10T14:00:00Z",
     "tenantId": "org-456",
     "traceId": "c4b...",
     "payload": { ... }
   }
   ```

---

## 3. Consumer Standard: Idempotency and Deduplication

Network partitions and retries guarantee **at-least-once delivery**. Duplicate events will arrive; consumers must handle them safely.

1. **MUST** implement consumer idempotency:
   - Check an `idempotency_keys` or `processed_events` table before executing side-effects.
   - Or design the state transition to be naturally idempotent (e.g., `UPDATE ... SET status = 'CANCELLED' WHERE status = 'PENDING'`).
2. **MUST NOT** acknowledge (commit offset / ACK) an event until its business processing and database changes have successfully committed.
3. **MUST** propagate and log the inbound `traceId` and `tenantId` across all consumer worker threads.

---

## 4. Retries and Dead-Letter Queue (DLQ) Standard

1. **MUST** separate transient errors from poison pills:
   - **Transient errors** (database timeout, rate limit, network blip): retry with exponential backoff and jitter (e.g., retry 3–5 times).
   - **Poison pills** (corrupted JSON, missing required fields, illegal state): route immediately to a Dead-Letter Queue (DLQ). Do not block partition processing in an infinite retry loop.
2. **MUST** set a maximum retry threshold before routing to DLQ.
3. **MUST** alert operations when messages land in the DLQ:
   - DLQ messages require root-cause analysis, bug fixes, and an audited replay script.
4. **MUST NOT** discard unprocessable events silently without logging and DLQ capture.

---

## 5. Schema Evolution and Compatibility

1. **MUST** design events to be backward-compatible:
   - Add only optional fields with sensible defaults.
   - Never remove or rename fields in an active event version.
   - Increment `eventVersion` when breaking schema changes are required.
2. **SHOULD** use a Schema Registry (e.g., Confluent Schema Registry with Avro/Protobuf/JSON Schema) to enforce schema compatibility at build and deploy time.
