# Workflow and State Machine Engine Standard

## Purpose

Complex domain entities in ERP, booking, procurement, and workflow systems go through explicit lifecycle transitions (e.g., `DRAFT` $\rightarrow$ `PENDING_APPROVAL` $\rightarrow$ `APPROVED` $\rightarrow$ `FULFILLED` or `REJECTED`).

Ad-hoc `if/else` checks scattered across controllers and services lead to invalid states, race conditions, and bypass of business rules. This standard defines the **Finite State Machine (FSM)** pattern for managing entity lifecycles predictably.

---

## 1. Core State Machine Principles

```text
               ┌───────────┐
               │   DRAFT   │
               └─────┬─────┘
                     │ submit() [Guard: items > 0]
                     ▼
             ┌───────────────┐
             │   SUBMITTED   │
             └───┬───────┬───┘
   approve()     │       │ reject() [Guard: reason required]
   [Role: ADMIN] │       │
                 ▼       ▼
          ┌──────────┐ ┌──────────┐
          │ APPROVED │ │ REJECTED │
          └──────────┘ └──────────┘
```

1. **Explicit States & Transitions:**
   - Every domain entity with a lifecycle must define an exhaustive list of valid states and permitted transitions.
   - Any transition not explicitly listed is illegal and must throw a `409 Conflict` or `422 Unprocessable Entity` business exception.
2. **Transition Guards:**
   - Before a state change occurs, a guard evaluates business preconditions (e.g., "Cannot transition to `APPROVED` if total > $10,000 without dual approval").
3. **Transition Actions & Side Effects:**
   - Pre-transition validation and state change occur inside the primary database transaction.
   - External side-effects (sending email, notifying webhooks) are published via the Outbox table in the same transaction.

---

## 2. Concurrency and Optimistic Locking

State transitions are high-contention operations prone to race conditions (e.g., two managers approving the same request at the same second).

1. **MUST** use Optimistic Locking:
   - Entities managed by a state machine must contain a `@Version private Long version;` column in JPA/Hibernate.
   - If two requests attempt to transition an entity concurrently, the first succeeds and the second fails safely with `OptimisticLockException` mapped to HTTP 409 Conflict.
2. **Atomic Transition Query (Alternative):**
   ```sql
   UPDATE bookings
   SET status = 'CONFIRMED', version = version + 1, updated_at = NOW()
   WHERE id = :id AND status = 'PENDING' AND version = :version;
   ```
   If rows updated == 0, the state has already changed; abort and return conflict error.

---

## 3. Permission and Tenant Verification per Transition

1. **Transition-Level Authorization:**
   - Permission to create a record does not imply permission to approve or cancel it.
   - Map individual permissions to transition actions:
     - `booking.submit`: customer or staff
     - `booking.approve`: manager or admin
     - `booking.override`: platform admin only
2. **Tenant Scoping:**
   - A state transition query must always include `WHERE id = :id AND tenant_id = :tenant_id`.

---

## 4. Audit Trail on State Changes

Every state transition must record:
- Entity ID and Type
- `previous_state` $\rightarrow$ `new_state`
- `transition_name` / `event`
- `actor_id` and timestamp
- Reason / comment (mandatory for cancellations and rejections)
