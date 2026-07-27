# SkyOps API Standards

## Purpose

This document defines platform-wide API standards for SkyOps. It is an architecture and governance document only. It does not define implementation code, controllers, handlers, database access, or generated server stubs.

These standards apply to public APIs, external customer APIs, internal service APIs, CLI APIs, extension APIs, AI APIs, audit APIs, notification APIs, topology APIs, and operational APIs.

## 1. API Philosophy

SkyOps is API-first. Every durable platform capability must be exposed through documented, versioned, secure, and observable contracts before UI, CLI, extension, AI agent, or automation consumers depend on it.

API design principles:

1. **Contract before consumer:** Define request, response, error, authentication, authorization, pagination, and lifecycle behavior before building clients.
2. **Tenant-safe by default:** Every API that touches customer data must require organization context and enforce authorization.
3. **Predictable over clever:** API names, URLs, response envelopes, pagination, filtering, and errors must be consistent across domains.
4. **Least privilege:** APIs expose only the data and operations required for the caller's permissions and use case.
5. **Explicit lifecycle:** Resource states, long-running operations, async jobs, and deprecations must be visible and documented.
6. **Observable by design:** APIs must propagate request IDs, correlation IDs, actor identity, tenant scope, and audit context.
7. **Backward compatible evolution:** Public and external APIs must avoid breaking clients and must use clear versioning and deprecation policy.
8. **AI-governed access:** AI APIs and tools use the same authentication, authorization, rate limiting, audit, and tenant isolation rules as human and automation clients.

## 2. API Versioning Strategy

SkyOps APIs use explicit major versions. Versioning is part of the URL for REST APIs and part of the protocol metadata for event streams, SDKs, and internal contracts.

Recommended REST version prefix:

```text
/api/v1
```

Versioning rules:

- Major versions represent compatibility boundaries.
- Additive fields are allowed within a major version when they do not change existing semantics.
- Breaking changes require a new major version or a documented migration and compatibility bridge.
- Deprecated fields and endpoints must include replacement guidance and removal timelines.
- SDK versions must declare the API major version they target.
- Event schemas and async job payloads must carry schema versions.
- Internal APIs may evolve faster but still require compatibility when multiple deployed service versions coexist.

Breaking changes include removing fields, changing field meaning, changing authorization requirements in a way that breaks valid clients, changing pagination behavior, changing error codes, changing resource identifiers, or changing idempotency semantics.

## 3. REST Standards

REST APIs are the default external API style for SkyOps. REST endpoints should model resources and state transitions clearly.

Rules:

- Use nouns for resources and avoid verb-heavy URLs.
- Use standard HTTP methods consistently.
- Use request bodies for create/update operations and query parameters for list shaping.
- Use status codes consistently with the SkyOps error and success standards.
- Make unsafe operations auditable and, when required, approval-gated.
- Keep transport concerns out of domain logic.
- Do not expose internal service boundaries or database structure directly.

Standard method semantics:

- `GET` retrieves resources and must be safe and idempotent.
- `POST` creates resources or starts non-idempotent operations unless protected by idempotency keys.
- `PUT` replaces a resource or declarative configuration where full replacement semantics are clear.
- `PATCH` partially updates a resource with documented merge semantics.
- `DELETE` deletes, revokes, disconnects, or archives a resource according to its lifecycle policy.

## 4. Naming Conventions

- Resource names use lowercase kebab-case in URLs.
- JSON field names use lower camelCase.
- Resource identifiers use `id` for SkyOps identifiers and explicit names such as `externalId`, `providerId`, or `digest` for external identifiers.
- Boolean fields use positive names such as `enabled`, `active`, `required`, or `verified`.
- Timestamp fields use clear UTC names such as `createdAt`, `updatedAt`, `deletedAt`, `startedAt`, `completedAt`, and `observedAt`.
- Enum values use lowercase strings with hyphens when needed.
- Avoid abbreviations unless they are accepted domain terms such as `api`, `cli`, `sso`, `scim`, `rbac`, `mfa`, `slo`, `mcp`, or `id`.

## 5. URL Structure

Canonical URL structure:

```text
/api/v1/{resource}
/api/v1/{resource}/{resourceId}
/api/v1/{resource}/{resourceId}/{subresource}
```

Tenant scoping rules:

- The authenticated organization context is normally selected through token claims, session context, request headers, or explicit organization selector.
- URLs should not include organization IDs on every endpoint unless the endpoint is naturally cross-organization or administrative.
- Project, environment, repository, deployment, cluster, and topology endpoints must validate that the resource belongs to the active organization.

Examples of resource-oriented URL families:

```text
/api/v1/projects
/api/v1/projects/{projectId}
/api/v1/repositories/{repositoryId}/branches
/api/v1/pipelines/{pipelineId}/runs
/api/v1/deployments/{deploymentId}
/api/v1/clusters/{clusterId}/namespaces
/api/v1/audit-logs
```

Nested URLs are allowed when the parent relationship is required to understand the resource. Avoid excessive nesting beyond two parent levels.

## 6. Authentication Flow

SkyOps supports browser sessions, API keys, service identities, SSO/OIDC flows, and short-lived tokens as approved by identity architecture.

Authentication flow requirements:

1. Client presents credentials or token.
2. API gateway or identity middleware validates token authenticity, issuer, audience, expiry, revocation, and session state.
3. Authenticated actor context is created with actor ID, actor type, organization memberships, session ID, scopes, and risk metadata.
4. Request context propagates actor, organization, request ID, and correlation ID downstream.
5. Sensitive authentication events produce audit and security records.

Authentication must never be bypassed for protected APIs. Local development exceptions must be isolated, explicit, and impossible to enable accidentally in production.

## 7. Authorization Flow

Authorization is evaluated after authentication and before resource access.

Authorization flow:

1. Resolve active organization and tenant context.
2. Resolve target resource and parent ownership without exposing cross-tenant details.
3. Evaluate RBAC permissions, resource ownership, environment protections, policy gates, and feature entitlements.
4. Deny by default when context is missing or ambiguous.
5. Record audit events for privileged, destructive, production-impacting, billing, security, and AI tool actions.

Clients must receive sanitized denial responses that do not reveal unauthorized tenant data.

## 8. RBAC Strategy

SkyOps RBAC is permission-based and assignable through roles to users, teams, service accounts, and API keys.

RBAC design rules:

- Permissions are atomic actions on resource types.
- Roles are named permission bundles.
- Role assignment may be scoped to organization, project, environment, repository, cluster, or application where supported.
- Built-in roles provide safe defaults; custom roles support enterprise needs.
- Effective permissions are computed from direct user roles, team roles, service identity scopes, API key scopes, and explicit policy constraints.
- High-risk permissions may require approval gates or step-up authentication.
- AI agents must never receive implicit elevated roles.

## 9. Error Response Standard

All APIs must return a consistent error envelope.

Conceptual error shape:

```text
{
  error: {
    code,
    message,
    details,
    requestId,
    correlationId,
    documentationUrl
  }
}
```

Error rules:

- `code` is stable, machine-readable, and documented.
- `message` is safe for the caller and must not include secrets or internal stack traces.
- `details` may include field validation errors or retry guidance when safe.
- `requestId` is always included when available.
- Authorization and not-found errors must avoid cross-tenant information leaks.
- Dependency failures must be classified without exposing provider credentials or raw internal errors.

Standard classes include validation error, authentication required, permission denied, not found, conflict, rate limited, idempotency conflict, dependency unavailable, operation pending, and internal error.

## 10. Success Response Standard

Success responses must be predictable and typed.

Single resource response:

```text
{
  data: { resource }
}
```

Collection response:

```text
{
  data: [ resources ],
  pagination: { ... },
  links: { ... }
}
```

Operation response:

```text
{
  data: { operation }
}
```

Rules:

- Include only fields authorized for the caller.
- Use consistent timestamps, status values, and relationship summaries.
- Avoid returning large nested graphs by default; use explicit include parameters or topology endpoints.
- For create operations, return the created resource or operation resource.
- For destructive operations, return final state or accepted operation state according to synchronicity.

## 11. Pagination Standard

Cursor pagination is the default for collection endpoints. Offset pagination may be used only for small administrative lists where ordering is stable and scale is bounded.

Pagination parameters:

- `limit`: requested page size within endpoint-defined bounds.
- `cursor`: opaque cursor returned by a previous response.

Pagination response metadata includes:

- `nextCursor` when more results are available.
- `hasMore` when the server can determine it.
- `limit` applied by the server.

Rules:

- Cursors are opaque and must not be constructed by clients.
- Default and maximum limits must be documented per endpoint family.
- Ordering used for pagination must be deterministic.
- Deleted or newly inserted records must not cause duplicate or missing results beyond documented consistency limits.

## 12. Filtering

Filtering uses query parameters with documented field names and supported operators.

Rules:

- Filters must be allowlisted per endpoint.
- Tenant and authorization filters are always enforced by the server and are not optional client filters.
- Avoid arbitrary database-field filtering in public APIs.
- Support common filters such as `status`, `type`, `environmentId`, `projectId`, `createdAfter`, `createdBefore`, `updatedAfter`, `updatedBefore`, and `ownerId` where relevant.
- Invalid filter fields or values return validation errors.

## 13. Sorting

Sorting uses a documented `sort` parameter.

Rules:

- Default sort order must be documented.
- Supported sort fields must be allowlisted.
- Prefix descending fields with `-`, such as `sort=-createdAt`.
- Sorts must be stable, deterministic, and index-aware for large collections.
- Sorting by expensive computed fields requires a materialized read model or explicit performance review.

## 14. Searching

Search is separate from filtering. Search may use full-text indexes or derived search services.

Rules:

- Use a `query` or `q` parameter consistently for keyword search.
- Document searched fields and matching behavior.
- Search results must be tenant-scoped and permission-filtered.
- Search endpoints must support pagination.
- Do not expose unauthorized resource existence through search ranking or counts.

## 15. Validation Strategy

Validation occurs at the API boundary and again at domain boundaries where business invariants require it.

Validation rules:

- Validate request shape, required fields, enum values, identifiers, timestamps, limits, and cross-field constraints.
- Return structured field-level validation errors.
- Normalize inputs consistently before persistence or downstream calls.
- Reject unknown fields for strict contract endpoints unless compatibility policy states otherwise.
- Treat provider payloads, webhooks, AI tool requests, and internal service calls as untrusted input.

## 16. Rate Limiting

Rate limits protect platform availability, tenant fairness, and security.

Rate limit dimensions:

- Organization.
- User.
- API key or service identity.
- IP address where appropriate.
- Endpoint family.
- AI/tool execution category.

Rate limit responses must include retry guidance when safe. Enterprise plans may have configurable limits, but no tenant may bypass abuse and safety controls.

## 17. Idempotency

Idempotency is required for create or action endpoints that clients may retry safely, especially deployments, infrastructure actions, billing operations, file uploads, and external provider mutations.

Rules:

- Clients provide an idempotency key for eligible unsafe requests.
- Idempotency scope includes actor, organization, endpoint, and request fingerprint.
- Repeated identical requests return the original result.
- Repeated requests with the same key and different payload return an idempotency conflict.
- Idempotency records have bounded retention.

## 18. WebSocket Strategy

WebSockets support real-time product experiences such as deployment progress, pipeline status, logs tailing, alert updates, AI assistant interactions, and topology changes.

Rules:

- WebSocket connections must authenticate and authorize at connection time and for subscribed resources.
- Subscriptions must be tenant-scoped and resource-scoped.
- Messages must carry type, version, timestamp, request/correlation identifiers, and resource context.
- Backpressure, reconnect, heartbeat, and authorization revocation behavior must be documented.
- WebSockets are not the authoritative system of record; clients must reconcile with REST resources.

## 19. Event Streaming

Event streaming supports asynchronous integrations, audit pipelines, workflows, topology projections, and AI context building.

Rules:

- Events must have stable names, schema versions, event IDs, tenant context, correlation IDs, actor context where applicable, and timestamps.
- Events are append-only facts and should not be mutated after publication.
- Consumers must be idempotent.
- Sensitive payloads must be minimized and classified.
- Event contracts must be documented and compatibility-reviewed.

## 20. API Gateway Responsibilities

The API gateway is responsible for cross-cutting API concerns, not domain business logic.

Responsibilities:

- TLS termination and secure transport policy.
- Request routing.
- Authentication verification or delegation.
- Rate limiting and abuse protection.
- Request ID and correlation ID management.
- Basic request size limits.
- CORS policy for browser clients.
- API version routing.
- Observability, access logs, and gateway metrics.
- Safe error normalization for gateway-level failures.

The gateway must not contain product-specific authorization decisions that belong to domain services or policy services.

## 21. Service-to-Service Communication

Service-to-service APIs must be authenticated, authorized, observable, and contract-driven.

Rules:

- Use mutual service identity with short-lived credentials where possible.
- Propagate actor context for user-initiated requests.
- Distinguish end-user actions from autonomous service actions.
- Use synchronous calls for low-latency request/response needs and events for asynchronous workflows.
- Avoid direct database access across domain boundaries.
- Apply timeouts, retries, circuit breakers, and structured errors.

## 22. Internal APIs

Internal APIs are consumed only by SkyOps services, workers, tools, or trusted internal clients.

Rules:

- Internal does not mean insecure.
- Internal APIs require authentication, authorization, versioning, and observability.
- Internal APIs may expose operational details not appropriate for public APIs but must still protect tenant data.
- Internal APIs must be documented at the contract level.

## 23. External APIs

External APIs are consumed by customer automation, partner integrations, and enterprise systems.

Rules:

- External APIs require stable versioning, documentation, SDK support where appropriate, and deprecation policy.
- External APIs must be rate limited and auditable.
- External APIs must avoid exposing internal implementation details.
- Backward compatibility is mandatory within a major version.

## 24. Public APIs

Public APIs are external APIs available broadly to customers or third-party developers.

Rules:

- Public APIs require OpenAPI documentation, examples, changelog entries, and compatibility review.
- Public APIs must use stable error codes and response envelopes.
- Public APIs must support SDK generation or hand-authored SDK parity.
- Public APIs must have security review before release.

## 25. SDK Strategy

SkyOps SDKs should be generated from OpenAPI contracts where practical and wrapped with idiomatic developer ergonomics where needed.

SDK requirements:

- Support TypeScript, Go, and Python according to approved monorepo strategy.
- Preserve API error codes and request IDs.
- Support pagination helpers, retry policy hooks, idempotency keys, and authentication helpers.
- Avoid hiding authorization or tenant-scope requirements.
- Version SDKs according to API compatibility.

## 26. CLI API Strategy

The SkyOps CLI consumes public or external APIs unless a documented internal API is approved for administrative use.

Rules:

- CLI output must map cleanly to API resources.
- CLI commands must support request IDs for support diagnostics.
- Destructive CLI actions require confirmation or explicit force flags.
- Long-running CLI actions should display operation status and support polling.
- CLI authentication must use secure token storage and revocation flows.

## 27. VS Code Extension API

The VS Code extension must use documented APIs with least-privilege scopes.

Rules:

- Extension APIs must avoid exposing production secrets or privileged operations by default.
- Authentication must support secure browser-based or device flows.
- Extension calls must be tenant-scoped and project-aware.
- Extension features should prefer read-only context unless the user explicitly initiates an action.
- API responses should support lightweight summaries suitable for editor experiences.

## 28. AI API

AI APIs expose governed AI assistant, recommendation, context, and tool execution capabilities.

Rules:

- AI APIs must authenticate the caller and authorize every context read and proposed action.
- AI APIs must disclose whether actions are read-only, recommended, approval-required, or executable.
- Tool execution requires explicit policy checks and audit logging.
- AI context retrieval must be tenant-scoped and permission-filtered.
- Prompt, conversation, recommendation, and tool-call metadata must respect retention policy.
- AI APIs must support explainability through cited source entities, confidence, policy state, and action rationale.

## 29. Audit APIs

Audit APIs expose immutable security and operational evidence to authorized users.

Rules:

- Audit APIs are read-optimized, heavily filtered, paginated, and exportable through approved workflows.
- Access to audit APIs is privileged and must itself be audited.
- Audit filters must include time range and may include actor, action, target type, result, project, environment, and correlation ID.
- Audit responses must not expose secrets or sensitive payloads beyond policy.

## 30. Health Check APIs

Health APIs support orchestration, load balancing, and operational diagnosis.

Standard health categories:

- Liveness: process is running.
- Readiness: service can receive traffic.
- Dependency health: service can reach required dependencies.
- Build/version metadata: service version, commit, and deployment metadata where safe.

Health APIs must not expose secrets or tenant data.

## 31. Metrics APIs

Metrics APIs expose operational and product metrics through authorized views.

Rules:

- Internal service metrics should use standard telemetry formats.
- Product metrics APIs must enforce tenant and authorization scope.
- High-cardinality metrics must be controlled.
- Metrics APIs should support time range, resolution, aggregation, and resource filters.
- Metrics responses must document freshness and retention.

## 32. Logging APIs

Logging APIs expose log metadata, search, and retrieval pointers, not necessarily raw storage internals.

Rules:

- Log access is tenant-scoped, permission-checked, and audited for sensitive logs.
- APIs must support time range, severity, source, trace ID, deployment, pod, container, and application filters.
- Logs must be redacted according to policy.
- Large log retrieval should use streaming, pagination, or signed short-lived download URLs.

## 33. Notification APIs

Notification APIs manage user notifications, team routing, delivery channels, and preferences.

Rules:

- Notification reads are scoped to recipient or authorized administrators.
- Preference updates are audited when they affect security, incident, or billing notifications.
- Delivery status APIs should not expose provider secrets or raw provider payloads.
- Notification list endpoints use cursor pagination and status filters.

## 34. File Upload APIs

File upload APIs support artifacts such as logs, SBOM files, support bundles, imports, and attachments.

Rules:

- Prefer direct-to-object-storage uploads using short-lived signed URLs when appropriate.
- Validate file type, size, checksum, tenant scope, and purpose.
- Scan uploaded files where security policy requires it.
- Store metadata in PostgreSQL and file bodies in object storage.
- Use resumable or multipart flows for large uploads.

## 35. Long Running Operations

Long-running operations must not block HTTP requests until completion.

Examples include deployments, infrastructure plans, drift scans, vulnerability scans, AI tool execution, imports, exports, and large topology refreshes.

Rules:

- Return an operation resource with status, timestamps, progress, target entity, actor, and result summary.
- Support polling and real-time updates where appropriate.
- Operations must be cancellable when safe.
- Operations must record audit events for privileged or production-impacting actions.

## 36. Async Jobs

Async jobs execute background work triggered by APIs, events, schedules, or system workflows.

Rules:

- Jobs must be idempotent or have idempotency protection.
- Job payloads must carry tenant, actor, correlation, and schema version context.
- Retry, dead-letter, timeout, cancellation, and poison-message behavior must be documented.
- Job status should be exposed through operation or domain-specific resources when user-visible.

## 37. Retry Strategy

Retry behavior must be safe, bounded, and documented.

Rules:

- Retry only transient failures.
- Use exponential backoff with jitter.
- Respect server-provided retry guidance.
- Do not retry non-idempotent requests unless protected by idempotency keys.
- Preserve correlation IDs across retries.
- Surface final failure with actionable error codes.

## 38. OpenAPI Documentation Standards

Public and external REST APIs must have OpenAPI documentation.

OpenAPI requirements:

- Every endpoint includes summary, description, authentication requirements, authorization notes, parameters, request body, response bodies, error responses, pagination behavior, rate limits, and examples.
- Schemas use clear descriptions, enum values, formats, nullability, and deprecation markers.
- Security schemes are documented centrally.
- Generated SDKs must be reproducible from committed OpenAPI specifications.
- OpenAPI changes must be reviewed for compatibility.

## 39. API Security

API security requirements:

- Enforce TLS for all external traffic.
- Authenticate every protected endpoint.
- Authorize every tenant-scoped resource access.
- Validate all inputs.
- Use rate limits and abuse detection.
- Protect against injection, SSRF, path traversal, insecure deserialization, replay attacks, CSRF for browser flows, and confused deputy risks.
- Avoid leaking secrets, stack traces, internal topology, or unauthorized resource existence.
- Audit sensitive operations.
- Apply secure defaults for CORS, cookies, sessions, and tokens.

## 40. Future GraphQL Considerations

GraphQL is not the default SkyOps API style. REST remains the primary public API model.

GraphQL may be considered later for read-heavy topology, dashboard composition, or internal UI aggregation if:

- Authorization can be enforced at field and node levels.
- Query depth, cost, and rate limits are robust.
- N+1 query risks are controlled.
- Schema evolution is compatible and documented.
- GraphQL does not bypass REST contracts required by external automation and SDKs.

Any GraphQL adoption requires an ADR.
