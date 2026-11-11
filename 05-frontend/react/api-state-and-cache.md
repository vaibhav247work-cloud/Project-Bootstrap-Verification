# API Client, Server State, Local State and Cache Standard

Use one platform-owned typed HTTP client plus a server-state query/cache layer. **TanStack Query** is the React default; another choice requires an ADR showing equivalent cache, invalidation, retry and pagination behaviour.

```text
feature hook -> feature API adapter -> platform HTTP client -> Spring API
```

Components do not use direct `fetch`, Axios or third-party SDK calls. The platform client owns base URL, headers, token attach, request cancellation, trace ID, Problem Details normalization and the documented 401 refresh path. It never silently retries unsafe commands or grants permission.

Use generated OpenAPI contracts/types where approved, otherwise runtime validate/map critical or untrusted responses. Map times, money, IDs and nulls deliberately; TypeScript itself is not runtime validation.

## Query/mutation policy

All query keys include inputs that change output; tenant data always includes tenant identity, e.g. `["tenant", tenantId, "booking", id]`. On tenant switch, logout, session invalidation or access change, cancel/remove affected data before display.

Set explicit freshness and retention per query. Use bounded cursor pagination for high-volume lists; cancel obsolete searches; prefetch only likely allowed data. Server state stays in query cache—not global client state.

All commands run through feature mutation hooks, with pending/double-submit prevention. Use backend idempotency keys for duplicate-harm create/payment/reservation commands. On success update/invalidate the smallest correct queries. Optimistic updates require safe rollback and are not default for irreversible actions. Never auto-retry non-idempotent mutations.

| Failure | Client rule |
| --- | --- |
| Network/temporary 5xx | Bounded query retry/backoff with recovery UI |
| 401 | One refresh path, else safe logout |
| 403 | Show denied; do not retry as login |
| 409 | Show business conflict and reload/resolve |
| 422 | Show safe field/action error without losing input |
| 429 | Stop retry storm; honour `Retry-After` if supplied |

A timeout does not prove a command failed; critical commands use idempotency/status recovery.

Local state is only ephemeral UI state. Keep it local, then lift, then feature store/context; a global store needs an ADR. Do not persist credentials. Persisted preferences/offline cache require an owner, schema version, tenant, TTL, size bound and eviction on logout/tenant switch.

Test mapping, query key isolation, mutation invalidation, double submit, 401/403/409/422/429, stale request cancellation and safe logging.
