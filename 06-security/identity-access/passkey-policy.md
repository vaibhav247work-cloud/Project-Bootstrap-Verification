# Passkey Policy

## Decision

Passkeys are an optional, PRD-enabled primary authentication method and can also satisfy a strong MFA requirement. They do not replace JWT access tokens, refresh-session rotation, logout, session expiry, tenant context, or authorization checks.

```text
Password login or passkey login
        ↓
verified user identity
        ↓
same JWT access token + rotating refresh session
        ↓
same RBAC / tenant / data-scope authorization
```

Passkeys use WebAuthn/FIDO public-key credentials. The server stores a public credential key, never a reusable shared password/passkey secret.

## Rules

1. **MUST** use a mature WebAuthn/FIDO library and validate challenge, relying-party ID/origin, signature, credential/user binding, and counter/credential metadata as applicable.
2. **MUST** require a recently authenticated session and reauthentication before adding or removing a passkey from an existing account.
3. **MUST** allow more than one registered passkey/device per user and provide credential naming/revocation.
4. **MUST** define account recovery before enabling passkey-only login; recovery is a high-risk flow and requires reauthentication/audit controls.
5. **MUST** keep existing password/SSO availability as the project policy requires; do not force a passkey-only experience without a PRD decision and recovery plan.
6. **MUST** audit registration, successful/failed use, rename, removal, and recovery of passkeys.
7. **MUST NOT** treat a passkey as authorization; roles, tenant context, data scope, and business-state checks remain server-side.

## PRD decisions

- Passkeys enabled: yes/no
- Optional login method, MFA method, or passwordless-required method
- Supported recovery methods
- Privileged-role/high-risk-action reauthentication policy
