# Authentication Policy

## Default decisions

| Area | Standard | Project-level choice |
|---|---|---|
| Primary login identifier | Email | Email remains the default; a different identifier needs an ADR |
| Account-creation capability | Self-registration, admin creation, invitation, import, SSO JIT provisioning, integration/API, and service accounts are supported patterns | Enable only the flows required by the PRD |
| Invitations | Supported but not mandatory | Decide per project |
| SSO | Optional | Decide providers/client OIDC or SAML requirements in the PRD |

## Email identity rules

1. **MUST** validate email on the backend and store a normalized value for unique login lookup.
2. **MUST** enforce the appropriate uniqueness scope. In a shared-identity product, email is globally unique; if a business requirement permits the same email in different organizations, document the account-linking and login-selection flow in an ADR.
3. **MUST** verify email ownership before enabling self-registered accounts or sensitive email changes.
4. **MUST NOT** treat email verification as authorization. Roles, organization, and data scope remain separate checks.

## Account creation methods

The platform supports these methods, but every PRD selects the enabled subset:

| Method | Use case | Default status after creation |
|---|---|---|
| Self-registration | Public/customer product | `invited` or pending verification until email confirmed |
| Admin-created | Internal ERP/controlled onboarding | `invited` or `active`, according to invitation/password policy |
| Invitation | Controlled onboarding by email | `invited`, expiring invite token |
| Bulk import | Initial employee/student/customer load | Project-defined review/activation flow |
| SSO JIT provisioning | Client identity provider creates user on first approved login | `active` only after identity/tenant/role mapping checks |
| Integration/API | Trusted upstream system creates accounts | Least-privilege service identity and audit required |
| Service account | Background system-to-system work | Non-human account; no interactive login by default |

## PRD gate

Before authentication implementation, the PRD must state:

- enabled account-creation methods;
- whether email verification is required;
- invitation expiry and resend policy, if invitations are enabled;
- chosen SSO providers/protocols, if any;
- JIT provisioning permitted or invitation/pre-assignment required;
- user activation/suspension/disable policy;
- who can create, activate, deactivate, and assign initial roles.

See `token-session-policy.md` for the default JWT, refresh-token, logout, revocation, and multi-device-session policy.

See `password-and-login-protection.md` for password, reset, email-verification, MFA, and login-abuse policy.

See `sso-oidc-policy.md` for PRD-driven SSO support.

See `passkey-policy.md` for optional WebAuthn/FIDO passkey support.
