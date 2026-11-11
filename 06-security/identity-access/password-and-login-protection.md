# Password and Login Protection Standard

## Password policy

1. **MUST** hash passwords with Argon2id where supported. Use bcrypt only when Argon2id/scrypt is unavailable or for legacy compatibility; never use reversible encryption, MD5, SHA-1, or plain SHA-256 for passwords.
2. **MUST** use the password-hashing library's unique salt and store the complete encoded hash/parameters, never the plaintext password.
3. **MUST** benchmark hashing cost on the target infrastructure and use at least the current security baseline; review it periodically and rehash on successful login when parameters become outdated.
4. **MUST** allow long Unicode passwords/passphrases. Default minimum: 12 characters; default maximum: 128 characters, unless a selected password library imposes a safer documented constraint.
5. **MUST NOT** require arbitrary composition rules such as one uppercase, one digit, one symbol, or periodic password changes without a compromise reason.
6. **SHOULD** block known breached/common passwords using a privacy-preserving lookup or approved local list where practical.
7. **MUST NOT** log, email, display back, or store a password outside the password-hash operation.
8. **MUST** enforce MFA for platform administrators, tenant/organization administrators, and users who perform financial, security-configuration, or role-management actions. A project that weakens this requirement for any administrative account **MUST** record the exception in an ADR with a named security owner and review date.
9. **SHOULD** make MFA available for all other users and encourage enrolment through onboarding and account-settings UX.

## Login-abuse protection

Use multiple signals; do not rely only on source IP, because attackers can rotate IPs and a shared IP can represent many legitimate users.

| Control | Default policy |
|---|---|
| Per-account failures | Progressive delay after failures; temporary 15-minute lock after 5 consecutive failures within 15 minutes |
| Per-IP/device rate limit | Apply gateway/application rate limit; exact threshold depends on traffic profile |
| CAPTCHA / extra challenge | Trigger after suspicious/repeated failures; never mandatory on the first normal login |
| Error response | Same generic response for unknown email, wrong password, disabled account, and locked account where safe to do so |
| Audit | Record outcome/reason category safely; never password or raw token |

1. **MUST** rate-limit login, registration, email verification, password reset, and SSO callback endpoints.
2. **MUST** make account-based protection resistant to denial of service: use temporary/progressive lockout, recovery path, and monitoring rather than permanent automatic lockout.
3. **MUST** return comparable public responses and timing for existing/non-existing accounts in login and reset flows to reduce account enumeration.
4. **MUST** reset failure counters only after a successful authenticated login or a defined secure recovery event.
5. **MUST** notify/record security-relevant lockouts and suspicious activity according to project policy.

## Password reset flow

```text
Forgot-password request
  -> generic response regardless of account existence
  -> rate limited
  -> send single-use opaque reset token only to verified email

Reset submission
  -> validate token, expiry, single-use status
  -> set new password hash
  -> revoke active refresh sessions
  -> audit event + optional notification
```

1. **MUST** use a cryptographically random opaque reset token and store only its hash.
2. **MUST** make tokens single use, short lived (default 15 minutes), and invalid after password change.
3. **MUST NOT** change/lock an account merely because a reset request was submitted.
4. **MUST** use a reset URL/page that prevents token leakage through referrers, logs, analytics, and browser history as far as technically practical.
5. **MUST** rate-limit reset requests and reset-token guesses.
6. **MUST** revoke existing refresh sessions after successful reset; allow the project to choose whether the current reset session starts a fresh login immediately.

## Email verification and change

Use the same secure single-use token pattern for verification. Verify the new email before replacing the old login email. Email verification proves control of the inbox; it does not itself grant any role or organization access.

## Required audit events

- password created/changed/reset success or failure (no password/token values)
- login success/failure/lockout/challenge
- password-reset request/consumption/expiry
- email verification and email change
- MFA enrollment/disable/recovery, if enabled
