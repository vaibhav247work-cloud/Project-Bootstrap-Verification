# Infrastructure, Network Security and Operational Access Standard

## Infrastructure as code and inventory

Infrastructure, network policy, DNS, TLS, registry, cloud permissions, storage, monitoring, backup and deployment manifests are managed as reviewed version-controlled infrastructure-as-code (IaC) or a documented equivalent with audit history. Manual console edits are emergency-only, time-bound and reconciled to source of truth.

Maintain an owner-tagged inventory of production assets: accounts/projects, regions, clusters/hosts, load balancers, domains/certificates, databases/tenant inventory, Redis, buckets, queues, secret stores, registries, monitoring/logging destinations and third-party integrations.

- CI validates IaC syntax, plan/diff, policy/security rules and drift according to risk before apply.
- IaC contains references to secrets, never their values. Scan repositories/artifacts for secrets and risky public exposure.
- Separate production account/project/network/credentials from lower environments. Do not make shared convenience networking a production access path.
- Tag assets with environment, product/service, owner, cost centre and data classification to support audit, incident response and lifecycle cleanup.

## Network segmentation and exposure

Use default-deny network posture. Public ingress is limited to documented endpoints such as CDN/web frontend/API gateway/load balancer. MySQL, Redis, queues, object storage administration, Actuator, metrics, internal services and management consoles are private by default.

```text
Internet -> WAF/CDN/load balancer -> frontend/API public ingress
                                  -> private application/worker network
                                  -> private MySQL / Redis / queue / object storage
```

- Explicit firewall/security-group/network-policy rules restrict source, destination, protocol and port. Egress is controlled for workloads that do not need arbitrary internet access.
- MySQL and Redis accept only trusted application/backup/operations network identities. They are never public internet services.
- Separate management access path (VPN/zero-trust/bastion or equivalent) from application traffic. Production admin interfaces do not share ordinary public user routes.
- Protect public edge with appropriate DDoS/CDN/WAF/rate-limit/bot controls based on product threat model. WAF is defense in depth, not a replacement for secure code/authorization.
- DNS changes, domains, redirects and external callback allow-lists are reviewed, monitored and protected against takeover/misrouting.

## TLS, certificates and HTTP edge

- Enforce HTTPS for all user/API traffic; redirect/disable insecure HTTP except narrowly documented validation/bootstrap cases. No mixed active content.
- Use current supported TLS protocol/cipher configuration, managed/approved certificates and secure private-key storage. Certificate renewal/expiry is monitored with actionable alert before expiry.
- Configure HSTS, secure cookie attributes, CSP, CORS, request size/time limits, compression and caching based on the frontend/API security standard.
- TLS terminates only at approved edge/load balancer; internal service encryption/mTLS is selected from risk/data/network model and documented. Certificate rotation/revocation is tested.

## Identity, IAM and operational access

Human and workload access follows least privilege, separation of duties, just-in-time/short-lived credentials where supported, strong MFA and auditable identity—not shared administrator accounts or static personal credentials.

| Access type | Required control |
| --- | --- |
| Developer | no standing production DB/cluster admin; approved scoped non-prod access |
| CI build | build/test/registry scope only; no broad production secret/deploy authority |
| CD/deploy | environment-scoped deploy role, protected approvals and audit trail |
| Application workload | dedicated service identity, only required secret/db/cache/bucket/queue actions |
| Database backup/restore | separate least-privilege role with guarded restore authorization |
| Production support | temporary approved read/scope access with session/log audit |
| Break-glass | named emergency path, MFA, dual/authorized approval where feasible, automatic expiry and post-use review |

- Review privileged access and dormant accounts regularly. Immediately revoke access on role change/offboarding/compromise.
- Store workload secrets in approved secret manager and prefer workload identity/federation over long-lived cloud keys where platform supports it.
- Production database changes use approved migration/operations process. Direct production SQL requires documented emergency/operational authorization, exact target confirmation, least privilege, audit and post-change validation.
- Never share real production dumps, SSH keys, kubeconfigs, cloud keys or client credentials through chat, email, tickets or source control.

## Kubernetes/container platform hardening (when used)

- Namespaces, service accounts, RBAC, network policies, resource quotas/limits and admission policies are scoped by environment/service.
- Workloads run non-root, avoid privileged mode/host network/host PID/hostPath mounts/capability escalation, use read-only root filesystem where feasible and have explicit resource limits.
- Image pull comes from approved registry/digest; admission verifies policy/signature/provenance where supported. Scan images and base images continuously.
- Kubernetes API, dashboard and metrics endpoints are private/authenticated. Cluster admin is tightly limited; application teams receive minimal namespace roles.
- Secret projection/mount/access is limited to required workload and rotated according to secret policy. Do not use ConfigMap/labels/annotations to carry secrets.

## Vulnerability, patch and configuration management

- Track OS/base-image/runtime/library/cloud-service vulnerabilities and configuration drift. Prioritize remediation by exploitability, public exposure, tenant/data impact and compensating controls.
- Define patch SLA by severity and exposure. Critical exploited/public-facing issues receive incident priority; exceptions are time-bound, approved and monitored.
- Rebuild/redeploy images for base/runtime patches rather than patching live containers. Validate health/security and rollback compatibility after patching.
- Periodically review ports, IAM roles, network policies, public buckets, TLS/certificates, WAF rules, audit logs and unused assets.

## Required verification

- IaC plan/policy/secret scan and drift review.
- External attack-surface scan validates only intended endpoints/ports/DNS/certificates are public.
- Network-policy/firewall test proves app can reach necessary services and an unauthorized source cannot.
- IAM access review, break-glass drill, service-account permission test and production access audit review.
- Container/Kubernetes hardening scan and TLS/security-header/certificate-renewal checks.
