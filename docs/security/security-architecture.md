# SkyOps Security Architecture

## Purpose

This document is the official security architecture blueprint for SkyOps. It translates the approved product, software, monorepo, engineering, database, API, backend, and frontend architecture decisions into security controls that must guide every future engineering decision.

SkyOps is an AI-powered platform engineering and DevOps operating system. It manages source repositories, CI/CD pipelines, artifacts, Kubernetes clusters, infrastructure automation, observability data, security findings, notifications, webhooks, CLI clients, editor extensions, and AI-assisted actions. Security must therefore be built as a platform property, not as a feature added after implementation.

## Security goals

SkyOps security architecture optimizes for:

- Strong tenant isolation across organization, workspace, project, environment, data, identity, network, runtime, logs, analytics, and AI context.
- Least privilege for users, services, agents, integrations, automation, and AI tools.
- Auditable administrative, operational, CI/CD, cluster, and AI-driven actions.
- Defense in depth across frontend, API, services, workers, events, storage, Kubernetes, and third-party integrations.
- Secure-by-default onboarding for enterprises using SSO, SCIM, MFA, RBAC, API keys, and compliance controls.
- Reliable incident response, forensics, recovery, and customer trust.

## 1. Security philosophy

SkyOps follows a security-first operating model:

1. **Assume compromise:** Every boundary must be authenticated, authorized, logged, rate-limited, and monitored.
2. **Authorize every action:** Authentication proves identity; authorization determines allowable behavior for the current tenant, resource, environment, and risk level.
3. **Tenant isolation is non-negotiable:** Tenant context must be explicit in requests, persisted records, events, caches, logs, metrics, search indexes, and AI retrieval.
4. **Security controls are layered:** No single mechanism is trusted alone. Identity, policy, schema validation, database scoping, network controls, runtime controls, and audit work together.
5. **Human and AI actions are governed equally:** AI suggestions and tool executions must obey the same permissions, approvals, policies, and audit trails as human actions.
6. **Evidence over claims:** Security decisions must produce evidence through logs, traces, audit events, policy results, and compliance reports.
7. **Secure defaults with enterprise flexibility:** Enterprise customers need configuration options, but unsafe behavior must never be the default.

## 2. Zero Trust architecture

SkyOps uses Zero Trust principles internally and externally.

Core rules:

- No request is trusted because it originates from a private network, internal service, worker, browser session, CLI, or extension.
- Every request carries authenticated identity and tenant context.
- Every service validates identity, authorization, request shape, tenant scope, and policy requirements before acting.
- Service-to-service traffic uses short-lived workload identity or mTLS where appropriate.
- Lateral movement is reduced through network policy, service permissions, separate credentials, runtime sandboxing, and scoped data access.
- Administrative actions require stronger controls such as MFA, approval, justification, and audit.

Zero Trust applies to web UI, API clients, backend services, workers, event consumers, Kubernetes controllers, CLI, VS Code extension, webhooks, and AI tool calls.

## 3. Authentication strategy

Authentication establishes a verifiable actor identity.

Supported actor classes:

- Human users.
- Organization administrators.
- Service accounts.
- Internal services and workers.
- CI/CD runners and deployment controllers.
- CLI clients.
- VS Code extension clients.
- Webhook senders.
- AI assistant tool executor identities.

Authentication methods:

- Enterprise SSO through OIDC and SAML where required.
- OAuth2/OIDC for third-party integrations and identity providers.
- MFA for sensitive users and actions.
- API keys or personal access tokens for automation, with strong scoping and expiration.
- Signed webhook verification for inbound provider events.
- Workload identity or mTLS for internal service authentication.

Authentication must never be treated as authorization.

## 4. Authorization strategy

Authorization is policy-driven and tenant-aware.

Every authorization decision must consider:

- Actor identity and type.
- Organization, workspace, project, and environment scope.
- Resource type and resource ownership.
- Action being requested.
- Role assignments and direct grants.
- Attribute context such as environment criticality, branch protection, cluster sensitivity, policy state, and time-bound access.
- Feature entitlement and plan where applicable.
- Required approvals or break-glass state.

The backend is authoritative for authorization. Frontend permission gates improve usability but are not security boundaries.

## 5. Multi-tenant isolation

Tenant isolation is enforced across all layers.

### Identity layer

- Users belong to organizations through memberships.
- Organization identity configuration controls SSO, SCIM, MFA requirements, and session policy.
- Cross-organization access requires explicit membership and role assignment.

### API layer

- Tenant scope is derived from authenticated context and explicit route/path/resource identifiers.
- APIs reject ambiguous or conflicting tenant context.
- Every request is authorized against the resolved tenant and resource.

### Service layer

- Services receive tenant context as part of request metadata and validate it before business logic.
- Services do not trust tenant IDs supplied only by clients.
- Cross-domain calls preserve tenant, actor, request ID, and authorization intent.

### Database and storage layer

- Every tenant-owned record includes tenant identifiers aligned with approved database architecture.
- Queries must include tenant predicates or use repository abstractions that enforce scoping.
- Object storage paths, keys, and encryption context include tenant scope.
- Backups, exports, indexes, and analytics preserve tenant boundaries.

### Event layer

- Events include tenant context, actor context where applicable, resource identifiers, schema version, and request ID.
- Consumers validate event scope before processing.
- Dead-letter queues and replay tooling preserve tenant separation.

### Cache and search layer

- Cache keys include tenant and authorization-relevant scope.
- Shared caches must not mix tenant data.
- Search indexes are tenant-partitioned or enforce filter-level isolation with defense-in-depth checks.

### AI layer

- Retrieval and tool context is limited to the user's tenant and permissions.
- AI prompts, embeddings, summaries, memory, and evidence bundles preserve tenant scope.
- AI tool calls must re-authorize before execution.

### Frontend layer

- URLs include organization/workspace context for deep links.
- UI hides or disables unauthorized resources and actions but still relies on backend enforcement.
- Tenant switches clear tenant-scoped caches, graph state, search state, and sensitive panels.

## 6. Organization security model

Organizations are the top-level customer security boundary.

Organization-owned security settings include:

- SSO and identity provider configuration.
- SCIM provisioning policy.
- MFA requirements.
- Session lifetime and idle timeout.
- Allowed domains.
- API key and token policy.
- Role and group mapping.
- Audit log retention.
- Data residency and encryption preferences where supported.
- Notification and webhook security settings.
- Feature entitlements and AI policy.

Organizations contain workspaces, projects, environments, repositories, integrations, clusters, users, groups, service accounts, and audit records.

## 7. User identity model

A user identity represents a human principal. A user may belong to multiple organizations, but organization membership determines access.

Identity attributes:

- Stable user ID.
- Verified email addresses.
- Display name and profile metadata.
- Linked identity provider subjects.
- Organization memberships.
- Group memberships.
- MFA enrollment state.
- Session and device metadata.
- Account lifecycle state.

The same email address must not imply access across organizations. Authorization must use membership and role assignments, not email matching alone.

## 8. RBAC design

RBAC is the baseline authorization model. It is extended by attributes, policies, and approvals for sensitive operations.

Role scopes:

- Organization.
- Workspace.
- Project.
- Environment.
- Repository.
- Cluster.
- Service account.

Role examples:

- Organization owner.
- Organization administrator.
- Security administrator.
- Platform administrator.
- Workspace administrator.
- Project maintainer.
- Developer.
- SRE/operator.
- Auditor.
- Billing administrator.
- Read-only viewer.
- AI operator.

Roles are collections of permissions. Permissions are action-oriented and resource-specific.

## 9. Permission model

Permissions use the form `resource.action` at a conceptual level. They are evaluated against scope and attributes.

Permission categories:

- Read inventory and metadata.
- Read sensitive data.
- Create, update, delete resources.
- Execute pipelines.
- Approve deployments.
- Roll back deployments.
- Manage clusters.
- Manage integrations.
- Manage secrets references.
- Manage security policies.
- View audit logs.
- Export data.
- Manage users and roles.
- Execute AI tools.
- Approve AI proposed actions.

High-risk permissions require explicit assignment and cannot be implied by broad viewer roles.

## 10. Service-to-service authentication

Internal services and workers authenticate with workload identities. Service identity must be distinct from human, automation, and AI actor identities.

Requirements:

- Short-lived credentials preferred over static secrets.
- mTLS or signed service tokens for service calls where architecture supports it.
- Audience-restricted tokens.
- Service identity bound to deployment environment.
- Minimal service permissions.
- Request context propagation for tenant, actor, trace ID, and request ID.
- Separate identities for services, workers, controllers, and job runners.

Services must authorize both the calling service and the original actor intent when processing user-initiated workflows.

## 11. API security

API security requirements:

- TLS required for all external API traffic.
- Authentication required except explicitly public endpoints.
- Authorization required for every protected endpoint.
- Strict request schema validation.
- Response shaping to avoid leaking unauthorized fields.
- Rate limiting and abuse protection.
- Idempotency keys for sensitive or repeatable mutations.
- Consistent error envelopes without leaking secrets.
- Request IDs and trace correlation.
- Audit events for security-relevant mutations.
- CORS restricted to approved origins.
- OpenAPI or contract definitions reviewed for security behavior.

## 12. Session management

Session management must minimize account takeover risk.

Requirements:

- Secure, HTTP-only, SameSite cookies for browser session material where possible.
- Short-lived access sessions and refresh through secure flows.
- Idle timeout and absolute lifetime configurable by organization policy.
- Session revocation on password reset, SSO deprovisioning, MFA reset, role removal, or suspected compromise.
- Device/session inventory for users and administrators.
- Step-up authentication for sensitive actions.
- Tenant switch must re-evaluate permissions and clear tenant-scoped UI caches.

## 13. JWT and refresh token strategy

JWTs may be used for short-lived access tokens, not long-lived trust.

Rules:

- Access tokens are short-lived and audience-restricted.
- Refresh tokens are rotated, revocable, and stored securely.
- Token claims include subject, issuer, audience, expiry, issued-at, token ID, actor type, and tenant context only when appropriate.
- Avoid overstuffing tokens with mutable permissions; permissions should be resolved or verified server-side.
- Key rotation is mandatory.
- Token replay is mitigated through rotation, binding where feasible, anomaly detection, and revocation lists.

## 14. OAuth2 and OIDC integration

OAuth2/OIDC integrations support identity providers, Git providers, cloud providers, registries, and other external services.

Requirements:

- Use Authorization Code with PKCE for public clients.
- Validate issuer, audience, nonce, state, signature, and token expiry.
- Store integration tokens encrypted and scoped.
- Use least-privilege scopes.
- Refresh and revoke tokens securely.
- Record integration changes in audit logs.
- Separate user-delegated tokens from organization integration credentials.

## 15. SSO strategy

Enterprise SSO supports centralized identity governance.

Requirements:

- Organization-specific identity provider configuration.
- Verified domain ownership before enforcing domain-wide login policy.
- JIT provisioning only when allowed by organization policy.
- SCIM preferred for lifecycle automation.
- Group-to-role mapping with safe defaults.
- Break-glass accounts governed by MFA, monitoring, and audit.
- Clear lockout recovery process.

## 16. MFA strategy

MFA is required for organization owners, administrators, security administrators, and high-risk operations. Organizations may require MFA for all members.

Supported factors should prioritize phishing-resistant methods such as WebAuthn/passkeys, with authenticator apps as a compatible option. SMS should not be the preferred factor for privileged accounts.

Step-up MFA is required for:

- Changing SSO, MFA, or SCIM settings.
- Managing organization owners or security administrators.
- Creating or revealing high-privilege API credentials.
- Approving production deployments where policy requires it.
- Executing high-impact AI-recommended actions.
- Exporting sensitive data.

## 17. API key management

API keys and personal access tokens are high-risk credentials.

Requirements:

- Keys have owner, scope, expiration, last-used metadata, and creation audit record.
- Keys are shown only once at creation.
- Stored key material is hashed or encrypted according to token type and verification needs.
- Prefixes identify key type without revealing secret material.
- Rotation and revocation are supported.
- Organization policy can restrict creation, maximum lifetime, scopes, and allowed users.
- Usage is logged with actor, tenant, scope, IP, user agent, and request ID.

## 18. Secret management

Secrets include credentials, tokens, private keys, webhook signing secrets, integration refresh tokens, registry credentials, cluster credentials, and encryption keys.

Rules:

- Secrets are stored only in approved secret managers or encrypted stores.
- Secrets are never committed, logged, sent to telemetry, exposed in errors, or rendered in UI after creation.
- Access to secrets requires explicit permission and audit.
- Runtime secrets are injected securely and scoped to the workload.
- Rotation procedures are defined for every secret class.
- Secret scanning runs on repositories, CI logs, artifacts, images, and configuration where applicable.

## 19. Encryption at rest

Encryption at rest is required for databases, object storage, queues, backups, search indexes, logs, and analytics stores that hold customer data.

Requirements:

- Provider-managed encryption is the minimum baseline.
- Customer-managed keys may be supported for enterprise plans where approved.
- Sensitive fields use application-layer encryption when database-level encryption is insufficient.
- Encryption context includes tenant or resource scope where supported.
- Key rotation and access logging are required.
- Backups and replicas are encrypted with equivalent controls.

## 20. Encryption in transit

All network traffic carrying SkyOps or customer data must be encrypted in transit.

Requirements:

- TLS for external web, API, webhook, CLI, and extension traffic.
- Strong TLS configuration and modern cipher suites.
- Internal service encryption through mTLS or platform-level encrypted networking.
- Certificate validation for outbound integrations.
- No plaintext credentials in URLs.
- Secure WebSocket or SSE channels for real-time updates.

## 21. Certificate management

Certificate management must be automated and auditable.

Requirements:

- Automated issuance, renewal, and rotation.
- Inventory of certificates, owners, domains, expiry, and trust chain.
- Alerts before expiry.
- Separate certificates for environments and tenants where architecture requires.
- Private keys stored securely and never logged.
- Revocation plan for compromised certificates.

## 22. Secure configuration management

Configuration must be explicit, reviewed, and safe by default.

Requirements:

- Environment-specific configuration is versioned where appropriate.
- Secrets are not stored in plain configuration files.
- Security-critical defaults fail closed.
- Configuration changes are reviewed, tested, and audited.
- Feature flags do not bypass authorization.
- Misconfiguration detection runs in CI and runtime posture checks.

## 23. Audit logging

Audit logging is mandatory for security-relevant events.

Audit events capture:

- Actor identity and actor type.
- Organization, workspace, project, environment, and resource scope.
- Action and outcome.
- Timestamp.
- Request ID and trace ID.
- Source IP and user agent when applicable.
- Before/after metadata where safe.
- Approval and policy decision context.
- AI involvement and tool execution details.

Audit logs are immutable from the perspective of ordinary users and services. Access to audit logs is itself audited.

## 24. Compliance considerations

SkyOps should be designed to support enterprise compliance programs such as SOC 2, ISO 27001, GDPR, and industry-specific customer requirements.

Architecture considerations:

- Data inventory and classification.
- Tenant data isolation.
- Access reviews and least privilege.
- Audit evidence retention.
- Encryption and key management.
- Vulnerability management.
- Incident response records.
- Vendor and subprocessor controls.
- Data subject request workflows where applicable.
- Secure SDLC evidence from CI/CD.

Compliance requirements must be implemented through controls that improve real security, not only documentation.

## 25. Rate limiting

Rate limiting protects availability, cost, and abuse surfaces.

Apply limits by:

- IP address.
- User.
- Organization.
- API key.
- Service account.
- Endpoint class.
- Integration provider.
- AI tool usage and token budget.

Sensitive endpoints such as login, token creation, webhook intake, AI execution, search, logs, and exports require stricter controls and abuse monitoring.

## 26. Input validation

All inputs are untrusted.

Validation requirements:

- Validate shape, type, range, size, enum, format, tenant ownership, and state transition.
- Reject unknown or unsupported fields where contracts require strictness.
- Validate file uploads before processing.
- Validate webhook signatures before parsing expensive payloads.
- Validate AI tool arguments against schemas and policy.
- Enforce maximum sizes for logs, manifests, prompts, queries, and uploaded artifacts.

## 27. Output sanitization

Outputs must avoid leaking sensitive information and must be safe to render.

Requirements:

- Redact secrets, tokens, credentials, private keys, and sensitive environment variables.
- Sanitize logs, markdown, AI responses, repository content, manifests, and user-supplied metadata before browser rendering.
- Return only fields the actor is permitted to see.
- Use safe error messages for users and detailed diagnostics only in protected logs.
- Apply tenant checks before exporting data.

## 28. CSRF protection

Browser-based state-changing actions require CSRF protection.

Requirements:

- SameSite cookie policy.
- CSRF tokens or equivalent protection for unsafe methods when cookie-based sessions are used.
- Origin and referer validation where appropriate.
- CORS restricted to trusted origins.
- Non-browser API clients use bearer credentials and are not exempt from authorization or rate limiting.

## 29. XSS protection

SkyOps renders high-risk content such as logs, markdown, AI output, repository metadata, manifests, annotations, and notification content. XSS prevention is mandatory.

Controls:

- Escape by default.
- Avoid unsafe HTML rendering.
- Sanitize any intentionally rendered rich text.
- Use Content Security Policy.
- Avoid inline scripts and unsafe eval.
- Validate URL schemes.
- Treat AI-generated content as untrusted.
- Isolate third-party embedded content where required.

## 30. SQL injection prevention

SQL injection prevention is required across services and workers.

Controls:

- Parameterized queries or approved query builders.
- No string-concatenated SQL with untrusted input.
- Repository abstractions that enforce tenant predicates.
- Least-privilege database users per service.
- Query logging with sensitive value redaction.
- Security tests for dynamic filtering and sorting paths.

## 31. Supply chain security

SkyOps must secure source, dependencies, builds, artifacts, and deployments.

Requirements:

- Protected branches and required reviews.
- Signed commits or attestations where adopted.
- Dependency pinning and lockfile review.
- Software Bill of Materials for released artifacts.
- Build provenance and artifact signing.
- Least-privilege CI credentials.
- Isolated build runners for sensitive workloads.
- No long-lived secrets in CI.
- Release promotion controls and audit.

## 32. Dependency scanning

Dependency scanning covers application packages, container images, base images, GitHub Actions or CI plugins, infrastructure modules, and transitive dependencies.

Rules:

- Scan on pull request, scheduled cadence, and release.
- Prioritize exploitable and reachable vulnerabilities.
- Block releases for critical issues according to policy.
- Track exceptions with owner, reason, compensating control, and expiration.
- Feed findings into security dashboards, notifications, and audit evidence.

## 33. Container security

Container security controls:

- Minimal base images.
- Non-root runtime users.
- Read-only filesystems where practical.
- Dropped Linux capabilities.
- Image vulnerability scanning.
- Image signing and verification.
- SBOM generation.
- No secrets baked into images.
- Runtime policy enforcement.
- Separate build and runtime stages.

## 34. Kubernetes security

Kubernetes security controls:

- Namespaces aligned with service and environment boundaries.
- Network policies restricting east-west traffic.
- Kubernetes RBAC least privilege.
- Service accounts per workload.
- Pod security standards.
- Admission policies for images, privileges, host access, and resource limits.
- Secrets from approved secret managers.
- Audit logs enabled.
- Runtime threat detection where available.
- Cluster access governed by SkyOps RBAC and customer policy.

## 35. Network security

Network security reduces attack surface and lateral movement.

Requirements:

- Public exposure limited to approved ingress points.
- WAF or equivalent protection for external HTTP surfaces.
- DDoS protection for production endpoints.
- Segmented internal networks.
- Egress controls for services and workers.
- Private connectivity options for enterprise integrations where supported.
- Webhook ingress isolated and rate-limited.
- Observability traffic secured and scoped.

## 36. Backup and disaster recovery security

Backup and recovery security protects data under failure and incident conditions.

Requirements:

- Encrypted backups.
- Tenant-aware restoration controls.
- Access to backups restricted and audited.
- Backup integrity validation.
- Regular restore tests.
- RPO and RTO documented by environment.
- Disaster recovery runbooks include identity, key, certificate, and audit restoration.
- Incident recovery avoids cross-tenant data exposure.

## 37. Incident response strategy

Incident response must be planned before incidents occur.

Phases:

1. Preparation.
2. Detection.
3. Triage.
4. Containment.
5. Eradication.
6. Recovery.
7. Customer communication where required.
8. Post-incident review.
9. Control improvement.

SkyOps must support rapid revocation of sessions, API keys, service credentials, webhooks, integration tokens, and AI tool permissions.

## 38. Security monitoring

Security monitoring detects suspicious behavior across the platform.

Signals:

- Authentication failures and anomalies.
- MFA changes.
- Privilege changes.
- API key creation and unusual usage.
- Cross-tenant access denials.
- Rate limit violations.
- Webhook signature failures.
- CI/CD and deployment anomalies.
- Cluster access anomalies.
- Secret scanning findings.
- AI tool misuse or prompt injection indicators.
- Unusual export, search, or log access patterns.

Alerts must be routed to security operations with severity, evidence, affected tenant, and response guidance.

## 39. Threat modeling

Threat modeling is required for new features and major changes.

Process:

- Define assets, actors, entry points, trust boundaries, and data flows.
- Identify threats using STRIDE or an equivalent method.
- Map mitigations to design controls.
- Define abuse cases and misuse cases.
- Review tenant isolation, authorization, logging, AI behavior, and failure modes.
- Record residual risk and owner acceptance.

The dedicated threat model document expands this architecture for major SkyOps components.

## 40. Security best practices for AI features

AI features must be treated as privileged, untrusted, and auditable automation surfaces.

Controls:

- AI retrieval is tenant-scoped and permission-filtered.
- Prompts and model outputs are considered untrusted.
- Tool calls require schema validation, authorization, rate limiting, and audit.
- High-risk actions require human approval and possibly step-up MFA.
- AI responses cite evidence and identify uncertainty.
- Prompt injection defenses are applied to logs, repository content, documentation, tickets, and user-provided text.
- AI memory and embeddings must not cross tenant boundaries.
- AI cannot bypass policy, approvals, or RBAC.
- AI usage is monitored for abuse, data exfiltration, and unsafe action patterns.

## Component-specific security design

### Organizations

Organizations define the strongest customer boundary. Organization security settings govern identity providers, sessions, MFA, role mappings, token policies, data retention, audit visibility, notifications, webhooks, and AI enablement.

### Projects

Projects provide authorization scoping for repositories, pipelines, deployments, environments, topology resources, and observability signals. Project membership must not leak data from sibling projects unless a higher scope role grants it.

### Git repositories

Repository integrations require least-privilege OAuth scopes, webhook signature verification, secret scanning, branch protection awareness, commit metadata validation, and safe rendering of repository content.

### CI/CD pipelines

Pipelines require isolated runners, scoped credentials, protected variables, log redaction, artifact signing, approval gates, environment protections, and audit for execution, cancellation, retry, promotion, and rollback.

### Container registries

Registry credentials are secrets. Image metadata, SBOMs, vulnerability findings, provenance, signatures, and promotion status must be protected by project and environment permissions.

### Kubernetes clusters

Cluster access must use short-lived credentials or scoped integrations. SkyOps must map platform permissions to cluster operations without granting broad static kubeconfig access. Cluster events, manifests, logs, and exec-like capabilities require strict authorization and audit.

### AI assistant

The AI assistant can summarize, investigate, recommend, and propose actions. It cannot execute high-impact actions without explicit authorization, policy checks, audit, and approval. AI evidence and retrieval must be tenant- and permission-scoped.

### Topology Graph

The topology graph must never receive resources the user cannot access. Graph queries, caches, overlays, search results, live updates, AI insights, details panels, and shared links all preserve tenant and RBAC enforcement.

### Notifications

Notifications may contain sensitive operational details. Delivery channels must respect recipient permissions, tenant boundaries, opt-in policy, redaction rules, and auditability for critical events.

### Webhooks

Inbound webhooks require signature verification, replay protection, rate limiting, provider allowlists where applicable, payload validation, and event deduplication. Outbound webhooks require signing, retry limits, tenant isolation, secret rotation, and delivery audit.

### CLI

The CLI uses OAuth device flow or equivalent secure login, short-lived tokens, scoped API keys for automation, secure local storage, explicit tenant selection, and clear confirmation for destructive actions.

### VS Code Extension

The VS Code extension uses secure OAuth/OIDC flows, minimal local credential storage, explicit workspace-to-project mapping, permission-scoped API calls, safe rendering of AI output, and no implicit access to repositories outside the user's authorized tenant.

## Trust boundaries

Major trust boundaries:

- Browser to SkyOps edge.
- CLI/VS Code extension to SkyOps API.
- Webhook provider to webhook ingress.
- API gateway to backend services.
- Backend service to backend service.
- Services to databases, caches, search, queues, and object storage.
- Event producers to event consumers.
- CI/CD runner to SkyOps control plane.
- SkyOps to customer Kubernetes clusters.
- SkyOps to Git providers, registries, cloud providers, and notification providers.
- AI orchestrator to tools, retrieval indexes, model providers, and action execution services.
- Admin/operator access to production systems.

Every boundary requires authenticated identity where possible, tenant context, authorization, input validation, output controls, telemetry, auditability, and failure isolation.
