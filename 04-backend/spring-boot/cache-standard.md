# Cache Standard

## Cache-aside default

MySQL is authoritative. Read path: cache lookup → DB on miss → cache bounded safe result. Write path: transaction commits in MySQL → evict/update related cache after commit.

1. **MUST** assign owner, key, TTL, invalidation trigger, sensitivity and outage behaviour to every cache entry.
2. **MUST** use tenant-scoped Redis keys when multiple tenant DBs share Redis.
3. **MUST** evict effective-access cache on user status, role, permission, override or membership change.
4. **MUST** use short TTLs for security-sensitive/session-derived data and never let stale cache grant access after revocation.
5. **MUST** prevent cache stampede for expensive common keys using bounded locking/request coalescing or stale-while-revalidate where correctness permits.
6. **MUST NOT** cache raw credentials, tokens, passwords, unrestricted PII, or failed authorization as a broad reusable response.
7. **MUST** treat Redis failure as cache miss/fallback or safe failure; never an authorization allow.
8. **MUST** monitor hit rate, evictions, memory, latency, key count and cache errors.
