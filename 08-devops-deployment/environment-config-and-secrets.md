# Environment, Configuration, Feature-Flag and Secrets Standard

## Core rule

Code is versioned and promoted unchanged. Configuration varies by deployment and is supplied outside the code artifact. A repository must be safe to share without exposing credentials.

```text
same immutable artifact
  + approved environment configuration
  + runtime secret references
  = local / SIT / UAT / production deployment
```

Never use a code branch, hand-edited production file, or a different manually built JAR/image as the mechanism for environment-specific behaviour.

## Environment model

| Environment | Purpose | Required separation |
| --- | --- | --- |
| Local | developer feedback | local/synthetic credentials and data only |
| CI ephemeral | automated build/test | short-lived scoped credentials and disposable resources |
| Dev/integration | team integration | separate domains, DBs, Redis, buckets and provider sandboxes |
| SIT/staging | production-like system/release verification | production-like topology/security, never production data/secrets |
| UAT | client/product acceptance | controlled accounts/tenants/data and restricted access |
| Production | live service | isolated account/project/network/credentials and audited access |

At minimum, production has its own database instances, Redis deployment/namespace, object storage, secrets, DNS/TLS, monitoring project/index and third-party production credentials. Non-production must not reach production databases or use production client secrets.

## Configuration contract

Every deploy-time value has a documented configuration key, type, default/required status, sensitivity, owner and validation rule. Spring Boot binds configuration through validated `@ConfigurationProperties`; React receives only explicitly public build/runtime configuration.

| Class | Examples | Rule |
| --- | --- | --- |
| Public runtime config | API base URL, build version, enabled public locale | may be shipped to browser; no secret |
| Service config | pool size, log level, timeout, cache TTL | externalized and validated; safe defaults |
| Secret | DB password, signing key, OAuth client secret, Redis credential | runtime secret reference only; never bundle/log/source |
| Feature flag | gradual rollout/temporary capability | owned, documented, monitored, expiry/removal date |

- Validate all required configuration at startup. A missing/invalid secret, tenant datasource config or critical URL fails startup/readiness safely rather than silently using fallback values.
- Keep a versioned `.env.example`/configuration reference with placeholders and descriptions; it contains no real credential or client data.
- Never put a secret in source, committed config, Docker image layer, frontend bundle, test fixture, shell history, screenshot, ticket, build log or exception message.
- Use a secret manager/platform secret store with encryption, access audit, least privilege, versioning and rotation support. Base64 encoding is not secret protection.
- Rotate secrets with overlap/grace where the protocol permits, test the rotation process in non-production, revoke compromised values, and record owner/last rotation/expiry policy.

## Spring profiles and runtime configuration

- Profiles express broad behaviour (`local`, `test`, `sit`, `uat`, `prod`); individual deploy values are explicit keys, not a growing maze of profile-specific copied files.
- The active profile is explicit at deployment. Production refuses debug/demo/test authentication, permissive CORS, sample credentials, broad actuator exposure or test data seed mode.
- Configuration changes use reviewed change control/GitOps/IaC or an audited platform configuration path. Manual emergency changes are documented, time-bound and reconciled back to the source of truth.
- Hikari tenant datasource limits, Redis endpoints/ACL, API timeouts, CORS origins, JWT issuer/audience, CSP, upload limits and retention periods are environment config with validation—not hard-coded values.

## React build/runtime configuration

- Any variable compiled into the browser is public. Use it only for non-sensitive identifiers/endpoints/feature configuration.
- Prefer a small validated runtime config document or server-injected configuration when one immutable frontend artifact must promote across environments. It is integrity-protected, contains no secrets and has a safe failure state.
- Frontend build exposes app version/commit identifier for support and telemetry, not source-map/secret information.
- Do not use client feature flags for authorization, tenant selection or secret distribution. Backend enforcement remains mandatory.

## Feature-flag lifecycle

Every flag has name, purpose, owner, default per environment, affected tenant/user segment, permission compatibility, rollback behaviour, telemetry/alert signal and removal deadline.

- Flags are evaluated safely and consistently. A flag service/config outage follows a documented default; it never expands access.
- Flags that change storage/schema/API behaviour require forward/backward compatibility and migration plan.
- Test both enabled and disabled paths, including configuration reload/cache propagation where supported.
- Remove stale flags and dead code after rollout; flags are temporary control points, not permanent architecture.

## Required deployment checks

- Configuration schema validation and secret-reference resolution before rollout.
- Production/non-production endpoint, credential, CORS, email/SMS/payment and tenant datasource separation test.
- Secret scan of source, image, artifact, logs and generated frontend build.
- Feature-flag inventory/review for each release and stale-flag removal review on cadence.
- Audit evidence for production configuration/secret access/change and tested rotation/compromise procedure.
