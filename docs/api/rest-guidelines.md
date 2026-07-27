# SkyOps REST Guidelines

## Purpose

This document provides REST-specific guidelines for SkyOps API designers. It complements the platform-wide API standards and does not include implementation code, controllers, handlers, database code, or generated stubs.

## Resource Modeling

REST resources should represent durable SkyOps concepts such as projects, repositories, pipelines, builds, deployments, environments, clusters, namespaces, pods, services, ingresses, alerts, notifications, AI conversations, and audit records.

Guidelines:

- Model resources around business and platform concepts, not database tables.
- Keep resource representations stable even if internal storage changes.
- Expose relationships through identifiers, summaries, links, or dedicated relationship endpoints.
- Avoid deep object graphs by default.
- Use operation resources for long-running or asynchronous actions.

## HTTP Method Guidelines

- `GET` is safe and retrieves resources or collections.
- `POST` creates resources, starts operations, or performs action-like transitions when resource modeling is insufficient.
- `PUT` replaces complete declarative resources where full replacement is clear.
- `PATCH` updates partial resource fields with documented semantics.
- `DELETE` soft-deletes, disconnects, revokes, archives, or removes according to resource policy.

Action endpoints should be rare. When needed, use clear subresources such as approvals, cancellations, retries, rollbacks, exports, or operations rather than arbitrary verbs.

## Status Code Guidelines

Use status codes consistently:

- `200` for successful retrieval or synchronous update with response body.
- `201` for resource creation.
- `202` for accepted long-running operations.
- `204` for successful action with no response body when appropriate.
- `400` for malformed or invalid requests.
- `401` for missing or invalid authentication.
- `403` for authenticated but unauthorized requests.
- `404` for not found or intentionally hidden unauthorized resources.
- `409` for conflicts, version conflicts, state conflicts, or idempotency conflicts.
- `422` for semantically invalid input when syntactically valid.
- `429` for rate limit violations.
- `500` for internal failures.
- `502`, `503`, and `504` for upstream or dependency failures where applicable.

## Request Headers

Standard request headers:

- `Authorization`: bearer token, session-derived token, or approved credential mechanism.
- `X-Request-Id`: caller-provided request identifier when supported.
- `X-Correlation-Id`: workflow or trace correlation identifier when supported.
- `Idempotency-Key`: required for eligible unsafe retryable requests.
- `Accept`: requested response media type.
- `Content-Type`: request body media type.

Headers must not be used to bypass explicit resource authorization.

## Response Headers

Standard response headers:

- Request ID.
- Correlation ID.
- Rate limit metadata where applicable.
- Deprecation and sunset metadata for deprecated endpoints.
- Pagination links where appropriate.
- Cache control directives for cacheable resources.

Sensitive tenant data must not be placed in response headers.

## Request Body Guidelines

- Use JSON for standard request bodies.
- Use explicit schemas for every request body.
- Reject invalid enum values, unknown identifiers, invalid timestamps, and malformed nested objects.
- Avoid allowing clients to set server-owned fields such as creation timestamp, update timestamp, actor, tenant, status transitions, or audit metadata unless specifically documented.
- For partial updates, document whether null clears a field, is ignored, or is invalid.

## Response Body Guidelines

- Use the standard `data` envelope for success.
- Use the standard `error` envelope for failures.
- Include relationship identifiers and small summaries where useful.
- Avoid returning secrets, raw credentials, provider webhook secrets, session digests, API key hashes, or internal stack traces.
- Use consistent timestamps and status strings.

## Collection Guidelines

Collection endpoints must support cursor pagination unless explicitly exempt.

Collection features may include:

- Pagination through `limit` and `cursor`.
- Filtering through allowlisted query parameters.
- Sorting through documented sort fields.
- Search through a dedicated query parameter when supported.
- Lightweight includes for common relationship summaries.

Collection endpoints must never return unbounded result sets.

## Relationship Guidelines

Relationships may be exposed in three ways:

1. **Identifier fields:** Resource includes related resource IDs.
2. **Summary fields:** Resource includes a compact related resource summary.
3. **Relationship endpoints:** Dedicated endpoint lists related resources.

Use relationship endpoints for large or independently paginated collections, such as repository branches, pipeline runs, cluster pods, deployment events, or audit records.

## Include and Field Selection

SkyOps may support controlled includes or sparse fieldsets for performance and UX.

Rules:

- Includes must be allowlisted.
- Included resources must be independently authorization-checked.
- Includes must have depth limits.
- Field selection must not bypass redaction or authorization.
- Expensive includes should be moved to dedicated endpoints or read models.

## Concurrency and State Transitions

APIs that update mutable resources should support optimistic concurrency where conflicts are likely.

Rules:

- Resource versions or update timestamps may be used for conditional updates.
- State transitions must validate current state.
- Conflicting updates return conflict errors.
- Long-running state transitions return operation resources.

## Idempotent Resource Creation

For client-retryable creation requests:

- Require an idempotency key.
- Store request fingerprint and result for a bounded period.
- Return the original result for exact retries.
- Return conflict for mismatched payloads using the same key.

## Bulk Operations

Bulk APIs are allowed only when there is a clear operational need.

Rules:

- Bulk operations must define maximum item counts.
- Partial success behavior must be explicit.
- Each item result should include status and error details.
- Bulk operations affecting production, billing, security, or access must be auditable.
- Large bulk operations should use async operation resources.

## Deprecation Guidelines

Deprecated endpoints, fields, enum values, and behaviors must include:

- Replacement guidance.
- Deprecation date.
- Sunset or removal date when known.
- Changelog entry.
- Usage monitoring plan where possible.

## REST Endpoint Family Guidance

Recommended top-level resource families include:

```text
/api/v1/projects
/api/v1/applications
/api/v1/repositories
/api/v1/pipelines
/api/v1/builds
/api/v1/deployments
/api/v1/environments
/api/v1/clusters
/api/v1/registries
/api/v1/images
/api/v1/monitoring
/api/v1/alerts
/api/v1/notifications
/api/v1/audit-logs
/api/v1/integrations
/api/v1/ai
/api/v1/topology
/api/v1/operations
```

These examples describe URL families only and do not define implementation handlers.

## Topology REST Guidelines

Topology APIs expose the SkyOps Application Topology Graph through resource-oriented views.

Example topology URL family:

```text
GET /api/v1/topology
GET /api/v1/topology/projects/{projectId}
GET /api/v1/topology/deployments/{deploymentId}
GET /api/v1/topology/clusters/{clusterId}
GET /api/v1/topology/nodes/{nodeId}
GET /api/v1/topology/pods/{podId}
```

Topology response design:

- Return graph nodes and edges as conceptual resources with type, ID, label, status, timestamps, and relationship metadata.
- Support scopes such as project, deployment, cluster, node, pod, application, environment, and domain.
- Support depth limits to prevent unbounded traversals.
- Support filters for environment, status, resource type, health, and time window.
- Support point-in-time reconstruction where historical data is available.
- Apply authorization to every node and edge.
- Redact or omit nodes the caller cannot access.
- Include freshness metadata because runtime inventory is observed asynchronously.

Topology APIs must not expose raw Kubernetes secrets, provider credentials, hidden tenant resources, or internal-only infrastructure details without explicit permission.
