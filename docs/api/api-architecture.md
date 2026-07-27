# SkyOps API Architecture

## Purpose

This document defines the complete API architecture for SkyOps. It is the platform-level API design guide and does not generate implementation code, controllers, handlers, database code, or migrations.

The architecture aligns with the approved Product Vision, Software Architecture, Monorepo Structure, Engineering Standards, and Database Architecture.

## API Architecture Overview

SkyOps exposes capabilities through a layered API architecture:

```text
Clients
  -> API Gateway
  -> API Boundary and Policy Layer
  -> Domain Services
  -> Events and Async Operations
  -> Derived Read Models and External Integrations
```

Primary clients include the web app, admin app, CLI, SDKs, VS Code extension, customer automation, partner integrations, AI agents, and internal platform services.

## API Surface Classification

### Public APIs

Public APIs are stable customer-facing APIs intended for broad use by customers, partners, SDKs, and automation. They require full OpenAPI documentation, compatibility guarantees, rate limiting, security review, and deprecation policy.

### External APIs

External APIs are customer-accessible but may be scoped to enterprise integrations, early-access features, or partner workflows. They still require documented contracts and compatibility management.

### Internal APIs

Internal APIs are used by SkyOps services, workers, and trusted internal tools. They require authentication, authorization, versioning, observability, and documentation but may expose internal operational concepts unavailable to public clients.

### Service-to-Service APIs

Service-to-service APIs coordinate domain services. They must use service identity, propagate actor and tenant context, respect domain ownership, and avoid direct cross-domain database access.

### Event APIs

Event APIs publish immutable facts about domain changes, workflow activity, audit-relevant behavior, topology changes, AI actions, and operational signals. Event streams are contracts and must be versioned.

## API Gateway Architecture

The API gateway provides shared edge responsibilities:

- TLS and transport security.
- API version routing.
- Authentication verification or delegation to identity services.
- Request routing to domain services.
- Rate limiting and abuse controls.
- Request size limits and basic protocol protections.
- Request ID and correlation ID assignment.
- Gateway access logs and metrics.
- CORS and browser API protections.
- Safe gateway-level error responses.

The gateway must not become a business logic layer. Domain authorization, tenant-specific rules, workflow state, and product decisions remain in domain services and policy components.

## Request Lifecycle

A standard API request follows this lifecycle:

1. Client sends request with authentication credentials and optional request/correlation IDs.
2. Gateway validates transport, version, size, basic limits, and authentication.
3. Gateway enriches the request with request ID, correlation ID, actor context, and route metadata.
4. Domain service resolves active organization and target resource scope.
5. Authorization and policy checks run before protected data access.
6. Request is validated against contract and domain invariants.
7. Domain use case executes synchronously or starts an async operation.
8. Domain emits audit records and events where required.
9. Response is returned with standardized success or error envelope.
10. Metrics, traces, logs, and audit data provide end-to-end observability.

## Authentication Architecture

SkyOps supports multiple authenticated client types:

- Browser users using secure session-based flows.
- API clients using scoped API keys or bearer tokens.
- Service identities using short-lived service credentials.
- Enterprise SSO users using OIDC/SAML-backed identity flows as approved.
- AI agents acting as a user or service identity with explicit authorization.

Authentication outputs a normalized actor context:

- Actor ID.
- Actor type.
- Organization memberships.
- Session or credential ID.
- Scopes.
- Authentication method.
- Risk metadata.
- Token expiry and revocation state.

## Authorization Architecture

Authorization is policy-driven and tenant-aware.

Core authorization inputs:

- Actor context.
- Active organization.
- Target resource type and identifier.
- Resource ownership hierarchy.
- RBAC permissions.
- Team membership.
- Environment protection rules.
- Subscription entitlements.
- Feature flags.
- AI tool policy where applicable.

Authorization outputs allow, deny, approval-required, or step-up-required decisions. Denials must be safe and must not reveal unauthorized tenant resources.

## API Contract Architecture

REST contracts are the primary public API contracts and are documented with OpenAPI. Event contracts are documented with schema versions. WebSocket messages are documented as versioned message contracts. SDKs must be generated from or verified against these contracts.

Contract requirements:

- Stable resource names and identifiers.
- Standard success and error envelopes.
- Explicit authentication and authorization notes.
- Pagination, filtering, sorting, and search behavior.
- Rate limits and idempotency requirements.
- Deprecation markers.
- Examples for common and failure cases.

## Domain API Boundaries

API boundaries align to SkyOps domains:

- Identity and access.
- Organizations and tenancy.
- Source repositories.
- CI/CD pipelines.
- Builds and artifacts.
- Deployments and releases.
- Environments.
- Kubernetes operations.
- Infrastructure automation.
- Observability and logging.
- Security and audit.
- Notifications.
- Billing and subscription.
- AI orchestration and recommendations.
- Integrations.
- Application topology.

No API may expose another domain's internal storage as a shortcut. Cross-domain views must be designed as composed read models or documented aggregation APIs.

## Long-Running Operation Architecture

Long-running actions return operation resources rather than blocking the initial API request.

Operation resource responsibilities:

- Identify the operation, actor, tenant, target, and correlation ID.
- Expose status, progress, timestamps, result summary, and error details.
- Support polling.
- Support WebSocket or event updates where appropriate.
- Support cancellation when safe.
- Preserve audit history for privileged and production-impacting actions.

Applicable workflows include deployments, rollbacks, infrastructure runs, cluster scans, imports, exports, vulnerability scans, AI tool execution, topology refreshes, and billing exports.

## Async Job Architecture

Async jobs are used for work that should not run in the request path.

Architecture requirements:

- Jobs carry tenant, actor, correlation, and schema version context.
- Jobs are idempotent or protected by idempotency keys.
- Retries are bounded with backoff and dead-letter handling.
- Job results are reflected in domain resources, operation resources, events, or audit records.
- Job execution does not bypass authorization or policy decisions made at request time.

## WebSocket Architecture

WebSockets provide real-time updates for interactive experiences.

Primary use cases:

- Pipeline run progress.
- Deployment rollout status.
- Live operational events.
- Alert updates.
- Log tailing metadata and streams where approved.
- AI assistant interactions.
- Topology change notifications.

WebSocket connections must authenticate, authorize subscriptions, enforce tenant scope, handle backpressure, and require clients to reconcile final state through REST APIs.

## Event Streaming Architecture

Event streams support workflow automation, audit processing, topology projections, AI context building, notification dispatch, and integration callbacks.

Event design requirements:

- Event ID.
- Event type.
- Schema version.
- Organization and resource scope.
- Actor context when applicable.
- Correlation ID.
- Occurred timestamp.
- Payload classification.
- Compatibility policy.

Events are not a replacement for query APIs. They are immutable facts consumed by asynchronous systems.

## Topology API Architecture

The topology API exposes the SkyOps Application Topology Graph without leaking implementation details or unauthorized infrastructure data.

### Topology Goals

Topology APIs must help users and automation answer:

- What applications and deployments exist in an organization or project?
- Which repositories, branches, pipelines, builds, images, and registries produced a deployment?
- Which clusters, nodes, pods, services, ingresses, domains, and applications are connected?
- What is current versus historical?
- What changed recently and who initiated it?

### Topology Resource Views

Topology data is exposed through scoped graph views:

```text
GET /api/v1/topology
GET /api/v1/topology/projects/{projectId}
GET /api/v1/topology/deployments/{deploymentId}
GET /api/v1/topology/clusters/{clusterId}
GET /api/v1/topology/nodes/{nodeId}
GET /api/v1/topology/pods/{podId}
```

These are design examples for the API surface and do not define handlers or implementation.

### Topology Response Model

Topology responses should conceptually include:

- `nodes`: typed resources such as developer, organization, project, repository, pipeline, build, image, registry, deployment, cluster, node, pod, service, ingress, domain, and application.
- `edges`: typed relationships such as owns, triggers, produces, stores, deploys, schedules, contains, selects, routes, exposes, and represents.
- `scope`: requested graph scope.
- `filters`: applied server-side filters.
- `freshness`: observation timestamp and inventory freshness.
- `pagination` or continuation metadata for large graphs.
- `warnings`: redaction, partial data, stale inventory, or permission-limited results.

### Topology Query Controls

Supported controls should include:

- Scope by project, application, deployment, cluster, node, pod, environment, domain, or repository.
- Depth limit.
- Resource type include/exclude filters.
- Current-state or point-in-time mode.
- Time window for historical edges.
- Health/status filtering.
- Search by name, digest, domain, commit SHA, or external provider identifier where authorized.

### Topology Security

- Every node and edge is authorization-filtered.
- Redacted nodes may be omitted or represented as inaccessible placeholders depending on product needs.
- Secrets, token values, raw credentials, hidden tenant infrastructure, and provider-sensitive details are never exposed.
- Topology export requires explicit permission and audit logging.
- AI access to topology uses the same policy and redaction rules as human access.

## AI API Architecture

AI APIs are policy-controlled interfaces for conversations, context retrieval, recommendations, and tool execution.

AI API categories:

- Conversation APIs.
- Recommendation APIs.
- Context APIs.
- Tool discovery APIs.
- Tool execution APIs.
- Evaluation and feedback APIs.

AI API rules:

- Read context only within caller permissions.
- Explain recommendations with source entity references.
- Separate recommendation from execution.
- Require approvals for production-impacting or privileged actions.
- Audit tool calls, context access, approvals, rejections, and outcomes.
- Respect retention and customer AI policy settings.

## Audit API Architecture

Audit APIs expose immutable evidence records to authorized users and systems.

Design decisions:

- Audit APIs are read-only except for controlled export workflows.
- Audit queries require time ranges for large datasets.
- Audit export is a long-running operation.
- Audit access is audited.
- Audit responses use stable actor, target, action, result, and correlation fields.

## Health, Metrics, and Logging API Architecture

Operational APIs are split by audience:

- Infrastructure health endpoints for orchestrators and load balancers.
- Internal service metrics for platform operators.
- Product metrics APIs for customer-visible operational insights.
- Logging APIs for tenant-scoped log search and retrieval.

Operational APIs must not leak secrets or cross-tenant data. Internal metrics may expose service internals only on protected networks or authenticated internal channels.

## File Upload Architecture

File uploads use metadata-first design.

Flow:

1. Client requests an upload session for a specific purpose.
2. API validates tenant, authorization, file constraints, and policy.
3. API returns a short-lived upload target or operation resource.
4. Client uploads file content to approved storage.
5. API finalizes metadata, checksum, scan status, and references.

File bodies do not belong in PostgreSQL. PostgreSQL stores metadata, ownership, scan state, checksum, and storage references.

## SDK, CLI, and Extension Architecture

SDKs, CLI, and VS Code extension clients are first-class API consumers.

- SDKs use OpenAPI contracts and preserve standard errors.
- CLI uses public/external APIs and operation polling.
- VS Code extension uses least-privilege project-scoped APIs.
- All clients must surface request IDs for troubleshooting.
- Client tooling must not rely on undocumented internal APIs unless explicitly approved.

## API Documentation Architecture

API documentation includes:

- OpenAPI specifications.
- Endpoint guides.
- Authentication and authorization guides.
- Error code reference.
- Pagination/filtering/sorting/search guide.
- WebSocket message contracts.
- Event schema catalog.
- SDK documentation.
- CLI mapping documentation.
- Topology API guide.
- AI API safety and tool execution guide.

Documentation is versioned with code and reviewed with contract changes.

## API Security Architecture

API security is enforced at multiple layers:

- Edge transport and gateway controls.
- Authentication and token validation.
- Authorization and tenant policy checks.
- Input validation and output redaction.
- Rate limiting and abuse detection.
- Audit logging.
- Secure service-to-service identity.
- Secret minimization.
- Dependency and supply-chain security.

No API surface may bypass these controls for convenience.

## GraphQL Position

REST remains the official API architecture for SkyOps. GraphQL may be evaluated later for constrained read-heavy aggregation scenarios, especially topology dashboards, but only through an ADR that proves field-level authorization, query cost control, compatibility, caching, observability, and SDK implications.

## Architecture Boundaries

This document intentionally does not define:

- Controller or handler code.
- Database tables or migrations.
- Backend repository implementations.
- UI workflows.
- Generated SDK code.
- Provider-specific integration implementations.

Future implementation must follow this architecture, the API standards, the REST guidelines, engineering standards, and approved ADRs.
