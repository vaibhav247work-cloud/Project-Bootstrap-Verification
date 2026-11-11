# Spring Boot HTTP Client and Connection-Pool Standard

## Default

Use one configured, reusable HTTP client per external integration—never construct a client per request. The application chooses:

| Situation | Default client |
|---|---|
| Traditional Spring MVC/blocking service call | `RestClient` backed by Apache HttpClient 5 connection pool |
| Non-blocking/high-concurrency or streaming call | `WebClient` backed by a named Reactor Netty `ConnectionProvider` |

Use one style consistently in a service. Do not call `block()` on a reactive event-loop thread.

## Per-integration client

```text
platform/integration/
  payment/
    PaymentClient.java
    PaymentClientConfig.java
    PaymentClientProperties.java
  sms/
    SmsClient.java
    SmsClientConfig.java
```

Each client has typed configuration and owns its base URL, authentication, pool limits, connect timeout, response timeout, retry policy, circuit-breaker policy, and safe logging/redaction rules.

## Mandatory rules

1. **MUST** use explicit connect, response/read, and pool-acquisition timeouts. No unbounded wait.
2. **MUST** bound total/per-route connections for each integration and size them from expected concurrency and vendor limits.
3. **MUST** reuse pooled keep-alive connections; do not create `RestClient`, `WebClient`, Apache client, or Reactor `ConnectionProvider` per request.
4. **MUST** use HTTPS, credentials from secure configuration, and certificate validation; never log authorization headers, API keys, or sensitive request bodies.
5. **MUST** retry only idempotent/safely idempotent operations with bounded backoff and jitter. Use an idempotency key for external write/payment calls when supported.
6. **MUST** apply circuit breaker/bulkhead behaviour for unreliable/slow integrations and return a safe business outcome.
7. **MUST** propagate request/correlation ID and tenant context only when the external service is authorized to receive it.
8. **MUST** collect latency, failure, timeout, retry, pool-saturation, and circuit-breaker metrics per integration.
9. **MUST** mock/stub integration tests and run contract/sandbox tests without real production secrets.

## Tenant-aware integrations

If credentials/configuration differ by tenant, resolve them from trusted server-side tenant configuration. Cache only safe values; never allow the browser to supply a third-party API credential or endpoint.
