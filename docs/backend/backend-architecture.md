# SkyOps Backend Architecture

## Purpose

This document defines the official backend blueprint for SkyOps. It is a design document only. It does not create implementation code, APIs, database migrations, handlers, controllers, or Go code.

The backend architecture aligns with the approved Product Vision, Software Architecture, Monorepo Structure, Engineering Standards, Database Architecture, and API Architecture.

## 1. Backend Philosophy

SkyOps backend systems must be secure, observable, resilient, domain-oriented, and suitable for enterprise SaaS scale. The backend must support thousands of organizations, millions of users, high-volume operational telemetry, AI-assisted workflows, and real-time application topology.

Backend principles:

1. **Domain ownership:** Each service owns a bounded context, data ownership rules, events, internal contracts, and operational responsibilities.
2. **API-first and event-driven:** Synchronous APIs serve request/response workflows; events publish durable facts for workflows, projections, audit, notifications, and AI context.
3. **Tenant isolation everywhere:** Every service must propagate organization, actor, resource, request, and correlation context.
4. **Secure by default:** Authentication, authorization, secret handling, audit, policy enforcement, and data minimization are mandatory.
5. **Operationally transparent:** Services must expose health, metrics, traces, structured logs, and actionable errors.
6. **Resilient by design:** Timeouts, retries, circuit breakers, idempotency, backpressure, graceful shutdown, and disaster recovery are part of the architecture.
7. **Topology-aware:** Backend services collaborate to maintain the SkyOps Application Topology Graph from developer activity through runtime exposure.
8. **Boring infrastructure, clear contracts:** Prefer proven cloud-native patterns and explicit contracts over clever hidden coupling.

## 2. Microservice Architecture

SkyOps uses a modular microservice architecture aligned to platform domains. Services may be independently deployable and should own their runtime, data access patterns, events, and operational dashboards.

Recommended primary services:

- API Gateway.
- Authentication Service.
- User Service.
- Organization Service.
- Project Service.
- Repository Service.
- Pipeline Service.
- Build Service.
- Deployment Service.
- Kubernetes Service.
- Container Registry Service.
- Monitoring Service.
- Logging Service.
- Notification Service.
- AI Service.
- Audit Service.
- Billing Service.
- Integration Service.
- Webhook Service.
- Scheduler Service.

Workers support asynchronous execution for pipelines, deployments, topology refresh, metric ingestion, log ingestion, notifications, vulnerability scans, policy evaluation, AI context building, billing aggregation, and cleanup tasks.

## 3. Service Responsibilities

Every service must have a clear responsibility boundary:

- Own domain validation and lifecycle rules.
- Own writes to its authoritative data.
- Publish events when state changes.
- Consume events from other domains only through documented contracts.
- Expose public or internal interfaces according to the approved API Architecture.
- Avoid direct writes to another service's data.
- Avoid synchronous dependency chains for workflows that can be event-driven.

## 4. Service Boundaries

Service boundaries are business boundaries, not technical layers.

Rules:

- Identity services own users, credentials, sessions, service identities, and authentication context.
- Organization and project services own tenant hierarchy and product ownership metadata.
- Repository, pipeline, build, registry, and deployment services own delivery lineage.
- Kubernetes, monitoring, and logging services own runtime inventory and operational signals.
- AI service consumes governed context and produces recommendations; it does not bypass domain services for actions.
- Audit service owns immutable evidence and must not become a general event store.
- Billing service owns commercial lifecycle and entitlements.
- Integration and webhook services own external provider connectivity and inbound provider events.

## 5. Internal Communication Strategy

SkyOps uses a hybrid communication model:

- **Synchronous internal APIs** for low-latency reads, authorization decisions, user-facing requests, and strongly consistent workflow steps.
- **Event bus** for durable state-change notifications, workflow fan-out, topology graph updates, audit enrichment, notifications, AI context updates, and eventual consistency.
- **Background jobs** for long-running work, retries, reconciliation, backfills, scans, imports, exports, and scheduled maintenance.

Internal communication requirements:

- Propagate tenant, actor, request ID, correlation ID, and causation ID.
- Use timeouts and bounded retries.
- Classify errors consistently.
- Prefer idempotent consumers and operations.
- Avoid direct cross-service database access.

## 6. API Gateway Design

The API Gateway is the front door for browser, CLI, SDK, extension, public API, and partner traffic.

Responsibilities:

- TLS termination and protocol enforcement.
- API version routing.
- Authentication verification or delegation.
- Request ID and correlation ID management.
- Tenant context propagation.
- Rate limiting and abuse protection.
- Request size limits and upload session routing.
- CORS and browser security policy.
- Routing to domain services.
- Gateway access logs, metrics, traces, and safe gateway errors.

The gateway must not contain domain business logic. Authorization policy enforcement may be coordinated at the gateway for coarse checks, but resource-specific authorization belongs to domain and policy services.

## 26. Background Worker Architecture

Workers process asynchronous work outside request paths.

Worker categories:

- Pipeline runners.
- Deployment controllers.
- Kubernetes inventory reconcilers.
- Topology projection builders.
- Metric ingesters.
- Log indexers.
- Notification dispatchers.
- Webhook processors.
- Vulnerability scanners.
- Policy evaluators.
- AI context builders.
- Billing aggregators.
- Retention and archival jobs.

Worker rules:

- Jobs carry tenant, actor, correlation, causation, and schema version context.
- Jobs are idempotent or protected by idempotency keys.
- Retries use exponential backoff with jitter.
- Poison messages go to dead-letter handling.
- Workers expose health, metrics, structured logs, and tracing.
- Workers never bypass authorization or approval decisions recorded by services.

## 27. Event Bus Architecture

The event bus carries durable domain facts and workflow signals.

Event principles:

- Events represent facts that already occurred.
- Events include event ID, type, schema version, tenant scope, actor context, resource identifiers, correlation ID, causation ID, occurrence time, and payload classification.
- Producers own event schemas.
- Consumers are idempotent.
- Sensitive data is minimized.
- Event contracts are versioned and compatibility-reviewed.

Event bus use cases:

- Delivery lineage updates.
- Topology graph projection.
- Audit evidence enrichment.
- Notification fan-out.
- AI context indexing.
- Webhook processing.
- Billing usage aggregation.
- Security and policy workflows.

## 28. Cache Strategy (Redis)

Redis is used for ephemeral acceleration and coordination, not authoritative state.

Approved uses:

- Session acceleration where compatible with identity architecture.
- Rate limit counters.
- Idempotency records with bounded TTL.
- Short-lived authorization decision caching with safe invalidation.
- Feature flag and entitlement cache.
- Distributed locks for narrow, time-bounded coordination.
- Work queue coordination where approved.
- Topology read-model hot cache for expensive graph views.

Rules:

- Cache keys include tenant scope when tenant data is involved.
- Cache entries have TTLs.
- Cache invalidation is event-driven where possible.
- Secrets are not cached unless explicitly approved and encrypted.
- Redis loss must degrade gracefully and never corrupt authoritative state.

## 29. File Storage Strategy

Object storage is used for large immutable or semi-structured files.

Stored artifacts may include:

- Build logs and artifacts.
- SBOM files and provenance documents.
- Export bundles.
- Uploaded support bundles.
- Archived audit partitions.
- Archived telemetry envelopes.
- AI evaluation artifacts.

Rules:

- PostgreSQL stores metadata, ownership, checksums, lifecycle state, and storage references.
- Uploads use short-lived signed URLs or mediated upload sessions.
- Files are encrypted, access-controlled, scanned when required, and lifecycle-managed.
- Object keys must not leak secrets or tenant-sensitive names where avoidable.

## 30. Secrets Management

SkyOps stores secret values in approved secret managers, not ordinary service databases.

Rules:

- Services store secret references, versions, fingerprints, and policy metadata.
- Secret access is least-privilege and audited.
- Secrets are rotated regularly and on compromise.
- Service identities use short-lived credentials where possible.
- Secret values are never logged, emitted in events, included in errors, or exposed to AI context.

## 31. Configuration Management

Configuration must be explicit, typed, validated, and environment-aware.

Requirements:

- Validate configuration at startup.
- Fail fast for missing required production configuration.
- Use safe defaults only for local development.
- Separate secret references from non-secret configuration.
- Support environment-specific overlays through approved deployment mechanisms.
- Emit configuration version metadata for diagnostics where safe.

## 32. Observability Strategy

Every backend service and worker must be observable.

Required signals:

- Structured logs with request ID, correlation ID, tenant where safe, actor where safe, operation, status, and error classification.
- Metrics for request rate, latency, errors, saturation, queue depth, retries, circuit breaker state, cache hit rate, and dependency health.
- Distributed traces across gateway, services, workers, event consumers, and external dependencies.
- Health checks for liveness, readiness, and dependency status.
- Audit records for sensitive state changes.

## 33. Error Handling

Error handling must be consistent across services.

Rules:

- Classify validation, authentication, authorization, not found, conflict, rate limit, dependency, timeout, cancelled, and internal errors.
- Preserve root cause internally while returning safe messages externally.
- Include request and correlation identifiers.
- Do not swallow errors.
- Do not expose secrets, stack traces, provider credentials, or unauthorized tenant details.
- Emit metrics and logs for error classes.

## 34. Retry Strategy

Retries are used only for transient failures.

Rules:

- Use exponential backoff with jitter.
- Respect idempotency and operation safety.
- Set maximum attempts and deadlines.
- Avoid retry storms through backpressure and circuit breakers.
- Preserve correlation IDs across retries.
- Send exhausted jobs to dead-letter or failure workflows.

## 35. Circuit Breaker Strategy

Circuit breakers protect services from cascading failures.

Use circuit breakers for:

- External provider APIs.
- Database replicas or noncritical read paths.
- Observability backends.
- AI model providers.
- Notification providers.
- Container registries and Kubernetes API servers.

Circuit state must be observable. Fallback behavior must be explicit and must not violate security or correctness.

## 36. Rate Limiting

Rate limiting protects availability and fairness.

Dimensions:

- Organization.
- User.
- API key.
- Service identity.
- IP address.
- Endpoint family.
- AI tool category.
- Webhook provider.

Rate limits are enforced at the gateway for edge traffic and inside services for domain-specific operations.

## 37. Service Discovery

Services discover each other through Kubernetes-native DNS, service mesh, or approved discovery mechanisms.

Requirements:

- Use stable service names.
- Support mTLS or equivalent service identity where approved.
- Avoid hardcoded network addresses.
- Health and readiness must control routing.
- Service discovery metadata must support environment isolation.

## 38. Scalability Strategy

SkyOps scales through horizontal service replicas, partitioned data, asynchronous workflows, read models, and tenant-aware workload isolation.

Strategies:

- Stateless API services scale horizontally.
- Workers scale by queue partition and workload type.
- High-volume telemetry uses specialized stores and short PostgreSQL retention.
- Read-heavy topology views use projections and caches.
- Large tenants may receive dedicated database or cluster placement.
- Backfills and reconciliations are rate-limited and tenant-aware.

## 39. High Availability Strategy

High availability requirements:

- Multi-replica services across availability zones.
- Readiness and liveness probes.
- Graceful shutdown and draining.
- Highly available PostgreSQL, Redis, event bus, and object storage dependencies.
- Queue-based buffering for temporary downstream outages.
- No single service instance should be required for critical path availability.

## 40. Disaster Recovery Strategy

Disaster recovery requires tested restoration of data, services, events, files, configuration, and secrets.

Requirements:

- PostgreSQL point-in-time recovery and restore drills.
- Object storage replication and retention.
- Secret manager backup or replication strategy.
- Event replay or durable event retention where required.
- Infrastructure-as-code rebuild capability.
- Runbooks for region failure, database corruption, accidental deletion, provider outage, and credential compromise.
- Defined RPO and RTO by environment and enterprise tenant tier.

## Backend Topology Graph Responsibility

The backend must power the SkyOps Application Topology Graph by connecting delivery events, artifact metadata, deployment state, Kubernetes inventory, ingress/domain exposure, monitoring signals, and audit actors.

Canonical graph path:

```text
Developer -> Repository -> Pipeline -> Build -> Image -> Registry -> Deployment -> Cluster -> Node -> Pod -> Service -> Ingress -> Domain
```

Collaboration model:

- Repository Service records repository, branch, commit, pull request, and developer contribution events.
- Pipeline Service records pipeline definitions, triggers, and run lifecycle events.
- Build Service records build runs, artifact outputs, provenance, and image production links.
- Container Registry Service records registry metadata, image digests, tags, SBOMs, signatures, and vulnerability summaries.
- Deployment Service records promotions, approvals, rollouts, rollback links, and environment/cluster targets.
- Kubernetes Service reconciles clusters, namespaces, nodes, pods, containers, services, ingresses, and runtime ownership references.
- Monitoring and Logging Services attach health, metrics, logs, traces, and alerts to topology nodes.
- Audit Service records actor, approval, and privileged operation evidence.
- AI Service consumes authorized topology context to explain, recommend, and assist without becoming the source of truth.
- Event bus and topology projection workers maintain current and historical graph projections in near real time.

The authoritative source of graph truth remains domain-owned records and events. Graph projections are derived, rebuildable, permission-filtered, and freshness-aware.
