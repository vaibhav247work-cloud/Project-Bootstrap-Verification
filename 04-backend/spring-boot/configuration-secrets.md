# Configuration and Secrets Standard

## Environments

Use explicit local, development, staging and production profiles. Code and schema remain equivalent; values differ through external configuration.

1. **MUST** bind configuration to typed `@ConfigurationProperties` with validation and fail startup for missing/invalid required configuration.
2. **MUST** keep URLs, pool sizes, timeout values, feature flags and integration settings outside source code.
3. **MUST** store passwords, signing keys, API keys and tenant DB credentials in a secrets manager or protected environment injection; never Git, logs or client apps.
4. **MUST** support secret/key rotation without source changes and define restart/reload process.
5. **MUST NOT** use production secrets/data in local or test environments.
6. **MUST** maintain a checked-in safe `.env.example`/configuration reference with variable description but no real values.
7. **SHOULD** use feature flags for controlled rollout; flags are not authorization.
