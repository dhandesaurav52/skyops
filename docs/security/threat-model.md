# SkyOps Threat Model

## Purpose

This document defines the baseline threat model for SkyOps. It identifies assets, actors, trust boundaries, major data flows, threats, mitigations, and component-specific security considerations. It supports the security architecture, authentication model, and RBAC model without creating APIs, schemas, or implementation code.

## Method

SkyOps uses STRIDE as the primary threat modeling framework:

- **Spoofing:** Impersonating users, services, integrations, or tenants.
- **Tampering:** Unauthorized modification of data, configuration, artifacts, pipelines, deployments, or AI context.
- **Repudiation:** Denying actions because audit evidence is missing or weak.
- **Information disclosure:** Exposing tenant data, secrets, logs, source content, vulnerabilities, or AI context.
- **Denial of service:** Exhausting APIs, workers, topology rendering, AI usage, logs, search, or integrations.
- **Elevation of privilege:** Gaining permissions beyond intended role, scope, tenant, service, or AI policy.

Threat modeling is required for new product capabilities, new trust boundaries, high-risk integrations, and major architecture changes.

## Critical assets

SkyOps protects:

- Organization identities, memberships, roles, and groups.
- Sessions, refresh tokens, API keys, service credentials, webhook secrets, integration tokens, and certificates.
- Tenant configuration, projects, environments, repositories, pipeline definitions, artifacts, and deployment history.
- Container image metadata, SBOMs, provenance, vulnerabilities, signatures, and registry credentials.
- Kubernetes cluster credentials, manifests, events, logs, metrics, topology data, and operational actions.
- Audit logs, security findings, compliance evidence, and incident records.
- AI prompts, retrieval context, embeddings, evidence bundles, recommendations, tool calls, and generated summaries.
- Notification destinations, webhook payloads, and delivery records.
- CLI and VS Code extension credentials and local context.

## Actors

Trusted and untrusted actor classes:

- Anonymous internet users.
- Authenticated users.
- Organization owners and administrators.
- Developers, SREs, auditors, and security administrators.
- Service accounts and API key users.
- Internal SkyOps services and workers.
- CI/CD runners and deployment controllers.
- Git providers, registry providers, cloud providers, notification providers, and identity providers.
- Webhook senders.
- CLI clients.
- VS Code extension clients.
- AI model providers and AI tool executors.
- Malicious insiders.
- Compromised customer accounts.
- Compromised dependencies, images, or build systems.

## Major trust boundaries

1. Public browser to SkyOps web edge.
2. CLI and VS Code extension to SkyOps API edge.
3. External webhook providers to webhook ingress.
4. Identity providers to authentication service.
5. API gateway to backend services.
6. Backend service to backend service.
7. Services to databases, caches, search indexes, object storage, and queues.
8. Event producers to event broker and consumers.
9. CI/CD runners to control plane and artifact stores.
10. SkyOps control plane to customer Kubernetes clusters.
11. SkyOps to Git providers, container registries, cloud providers, and notification providers.
12. AI orchestrator to model providers, retrieval indexes, tools, and action execution services.
13. Operators and administrators to production infrastructure.

Every boundary requires explicit identity, tenant context, authorization, validation, monitoring, and failure isolation proportional to risk.

## Tenant isolation threats

### Threats

- Cross-tenant data returned by API query bugs.
- Shared cache keys leaking data between organizations.
- Search index filters omitted or bypassed.
- Event replay processing another tenant's data.
- Object storage paths guessed across tenants.
- AI retrieval returning another tenant's context.
- Topology graph live updates leaking unauthorized resources.
- Logs, metrics, or traces queried without tenant predicates.

### Mitigations

- Mandatory tenant predicates in repositories and query abstractions.
- Tenant-scoped cache keys and invalidation.
- Tenant-partitioned or strictly filtered search indexes.
- Event schemas with tenant context and consumer validation.
- Object storage scoped keys, encryption context, and authorization checks.
- Permission-filtered AI retrieval and embeddings.
- Graph data APIs that omit unauthorized nodes and overlays.
- Tests that exercise cross-tenant denial paths.
- Audit and anomaly detection for cross-tenant access denials.

## Authentication threats

### Threats

- Session hijacking.
- Refresh token theft or replay.
- OAuth/OIDC state or nonce bypass.
- Weak MFA enrollment or reset flows.
- API key leakage.
- Webhook signature spoofing.
- Service identity impersonation.

### Mitigations

- Secure cookies, token rotation, revocation, and short lifetimes.
- Strict OAuth/OIDC validation.
- MFA for privileged users and sensitive actions.
- API key scoping, expiration, hashing/encryption, and usage monitoring.
- Signed webhooks with replay protection.
- Workload identity or mTLS for services.
- Authentication anomaly monitoring.

## Authorization threats

### Threats

- Horizontal privilege escalation across projects or environments.
- Vertical privilege escalation through overbroad roles.
- Client-side authorization bypass.
- Worker executing events without reauthorization.
- AI tool execution using hidden superuser privileges.
- Shared links bypassing permissions.

### Mitigations

- Server-side permission checks for every protected action.
- Scoped RBAC plus attribute and policy constraints.
- Reauthorization before mutations and high-risk operations.
- Worker validation of tenant, actor, and action intent.
- AI execution bound to user permissions and tool policy.
- Shared links resolve through normal authorization.
- Deny-by-default behavior.

## API and application threats

### Threats

- Injection through filters, search, manifests, logs, or AI prompts.
- XSS through rendered repository content, logs, markdown, or AI output.
- CSRF against browser sessions.
- CORS misconfiguration.
- Mass assignment of protected fields.
- Sensitive data in error messages.
- Rate-limit bypass.

### Mitigations

- Strict input validation and schema enforcement.
- Output encoding and sanitization.
- CSRF protections for cookie-based sessions.
- Restricted CORS.
- Explicit field allowlists for mutations.
- Safe error envelopes and redaction.
- Multi-dimensional rate limiting.

## Supply chain threats

### Threats

- Malicious dependency update.
- Compromised CI action or plugin.
- Poisoned container base image.
- Artifact tampering.
- Secret leakage in build logs.
- Untrusted pull request executing privileged jobs.

### Mitigations

- Dependency scanning, lockfile review, and update policy.
- CI plugin pinning and review.
- SBOMs, provenance, image signing, and verification.
- Isolated runners and least-privilege CI credentials.
- Log redaction and secret scanning.
- Protected branches and approval gates.

## Kubernetes threats

### Threats

- Stolen kubeconfig or cluster token.
- Overprivileged service account.
- Pod privilege escalation.
- HostPath or host network abuse.
- Lateral movement between namespaces.
- Exposed dashboard or control-plane endpoint.
- Unreviewed manifest applying dangerous permissions.

### Mitigations

- Short-lived cluster access and scoped integrations.
- Kubernetes RBAC least privilege.
- Pod security standards and admission policies.
- Network policies.
- Runtime detection and audit logs.
- Manifest policy evaluation before apply.
- No broad static kubeconfig distribution.

## CI/CD threats

### Threats

- Pipeline definition tampering.
- Malicious build artifact.
- Deployment approval bypass.
- Environment secret exfiltration.
- Runner breakout.
- Log poisoning or secret exposure.

### Mitigations

- Protected branches and change review.
- Artifact signing, provenance, and SBOMs.
- Approval gates and separation of duties.
- Scoped environment credentials.
- Isolated runners and sandboxing.
- Secret masking and log sanitization.
- Audit for every pipeline and deployment action.

## Git repository threats

### Threats

- Overbroad OAuth scopes.
- Spoofed webhooks.
- Repository metadata leakage.
- Secret committed to source.
- Branch protection bypass.
- Malicious pull request influencing automation.

### Mitigations

- Least-privilege provider scopes.
- Webhook signature and replay validation.
- Permission-scoped repository views.
- Secret scanning.
- Branch protection awareness.
- CI policy for untrusted pull requests.

## Container registry threats

### Threats

- Registry credential theft.
- Pulling unsigned or vulnerable images.
- Tag mutation causing unexpected deployment.
- SBOM or provenance tampering.

### Mitigations

- Secret management for registry credentials.
- Image signing and verification.
- Digest-based deployment references where possible.
- Vulnerability scanning and policy gates.
- Registry access audit.

## Topology Graph threats

### Threats

- Graph reveals hidden resources through edges or aggregates.
- Live updates leak unauthorized topology changes.
- Metrics overlays expose sensitive operational details.
- Shared graph URLs bypass permissions.
- Browser freezes from oversized graph responses.
- Context menu actions bypass server authorization.

### Mitigations

- Permission-filtered graph APIs.
- Redacted or aggregated unauthorized relationships.
- Overlay-specific permissions for metrics, logs, security findings, and deployments.
- Shared views resolved through normal RBAC.
- Progressive loading, node limits, clustering, and rate limiting.
- Server-side reauthorization for every graph action.

## AI Assistant threats

### Threats

- Prompt injection from logs, repositories, tickets, documentation, manifests, or user text.
- Cross-tenant retrieval leakage.
- AI hallucination causing unsafe action.
- Tool execution privilege escalation.
- Sensitive data sent to model providers without policy.
- AI memory retaining unauthorized context.
- Data exfiltration through summaries.

### Mitigations

- Treat all retrieved content and model output as untrusted.
- Tenant- and permission-filtered RAG.
- Evidence citations and confidence labels.
- Tool schemas, policy checks, RBAC checks, rate limits, and audit.
- Human approval for high-impact actions.
- Data minimization and provider policy controls.
- No cross-tenant AI memory or embeddings.
- Output redaction and monitoring for exfiltration patterns.

## Notifications threats

### Threats

- Sensitive data sent to unauthorized channels.
- Notification recipient no longer has access.
- Webhook notification endpoint compromised.
- Alert fatigue hides real incidents.

### Mitigations

- Permission-aware recipient resolution.
- Redaction profiles per channel.
- Signed outbound webhooks.
- Revalidation before delivery.
- Delivery audit.
- Severity routing and suppression governance.

## CLI threats

### Threats

- Token stored insecurely on developer machine.
- Wrong tenant targeted by command.
- Destructive commands run accidentally.
- Automation key over-scoped.

### Mitigations

- OS credential storage where possible.
- Explicit active organization/workspace/project context.
- Confirmation and dry-run for destructive actions.
- Scoped API keys with expiration.
- Audit with CLI client identity and version.

## VS Code extension threats

### Threats

- Extension token theft.
- Workspace confused with wrong SkyOps project.
- AI output rendered unsafely.
- Unauthorized repository metadata access.

### Mitigations

- Secure OAuth flow and protected local storage.
- Explicit project binding.
- Safe rendering and sanitization.
- Permission-scoped API calls.
- Minimal telemetry and redaction.

## Denial-of-service threats

### Threats

- Login brute force.
- Expensive global search queries.
- Log and metrics query overload.
- Topology graph expansion overload.
- Webhook floods.
- AI token exhaustion.
- Large file uploads.

### Mitigations

- Rate limiting by IP, user, organization, key, and endpoint class.
- Query cost limits and pagination.
- Time-window limits for logs and metrics.
- Graph size limits, clustering, and progressive loading.
- Webhook queue isolation and deduplication.
- AI budgets, quotas, and cancellation.
- Upload size limits and scanning queues.

## Repudiation threats

### Threats

- User denies approving deployment.
- Service account action lacks owner.
- AI action lacks evidence of human approval.
- Webhook event cannot be tied to provider delivery.

### Mitigations

- Immutable audit logs.
- Actor, request, tenant, resource, source, and outcome recorded.
- Approval records with policy and MFA context.
- API key and service account ownership metadata.
- Webhook event IDs, signatures, and delivery records.
- AI tool execution audit with prompt, evidence references, policy result, and approver where safe to retain.

## Threat modeling governance

Every feature threat model must document:

- Assets.
- Actors.
- Entry points.
- Trust boundaries.
- Data classification.
- STRIDE threats.
- Mitigations.
- Residual risks.
- Audit events.
- Abuse cases.
- Testing requirements.
- Security owner approval.

Threat models must be revisited when data flows, integrations, permissions, AI tools, or deployment environments change.
