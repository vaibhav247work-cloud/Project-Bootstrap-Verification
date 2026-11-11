# SSO / OIDC Policy

SSO is optional and enabled only when the client PRD requires Google, Microsoft, or the client's identity provider.

## Standard

1. **MUST** prefer OpenID Connect (OIDC) for new web/mobile SSO integrations. Use SAML only when required by the client identity provider.
2. **MUST** use the provider's authorization-code flow with PKCE where applicable; do not accept an unverified client-provided identity/email as proof of login.
3. **MUST** validate issuer, audience/client ID, signature/keys, expiry, nonce/state, and redirect URI according to the selected protocol/library.
4. **MUST** link external identities to the internal `users` record. SSO changes authentication only; the same RBAC, tenant scope, status, and audit rules continue to apply.
5. **MUST** decide in the PRD whether just-in-time (JIT) provisioning is allowed. If not, a user must be invited/pre-assigned before their first SSO login.
6. **MUST** map client-provider organization/groups to internal tenant/role policy only through an explicit reviewed mapping. Do not automatically trust any arbitrary external group as admin access.
7. **MUST** audit SSO login, identity link/unlink, provisioning, group/role mapping changes, and failures.
8. **MUST** retain a secure recovery/admin-access process if the client SSO provider is unavailable or misconfigured.

## PRD decisions

- Provider and protocol: Google/Microsoft/custom OIDC/SAML
- Single tenant or multiple client identity providers
- JIT provisioning: allowed / invitation-only
- Domain/group restrictions
- External group to internal role mapping
- Account linking and recovery policy
- SSO-only or password fallback allowed
