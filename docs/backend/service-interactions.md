# SkyOps Backend Service Interactions

## Purpose

This document explains how SkyOps backend services collaborate through synchronous calls, events, workers, caches, storage, and topology projections. It is a design document only and contains no implementation code.

## Interaction Principles

- Prefer domain events for fan-out and cross-domain reactions.
- Use synchronous calls only when the caller needs immediate, strongly scoped information.
- Propagate tenant, actor, request ID, correlation ID, and causation ID across every boundary.
- Do not use direct cross-service database writes.
- Make every long-running workflow observable through operation state and events.
- Make consumers idempotent.

## Standard Request Interaction

```text
Client -> API Gateway -> Domain Service -> Domain Storage
                              |
                              -> Audit Service
                              -> Event Bus -> Workers/Projections/Notifications/AI
```

Design decisions:

- The gateway handles edge concerns.
- The domain service owns validation, authorization, state transition, and event publication.
- Audit records are emitted for sensitive actions.
- Events allow other services to react without coupling the request path to every downstream concern.

## Authentication and Authorization Interaction

```text
Client -> API Gateway -> Authentication Service -> User/Organization context
Client -> Domain Service -> Authorization policy evaluation -> Resource access
```

Flow:

1. Gateway verifies authentication or delegates verification.
2. Authenticated actor context is attached to the request.
3. Domain service resolves tenant and target resource.
4. Domain service evaluates RBAC, policy, entitlement, environment protection, and resource ownership.
5. Denied requests return safe errors and may produce audit/security signals.

## Repository to Pipeline Interaction

```text
Git Provider -> Webhook Service -> Repository Service -> Event Bus -> Pipeline Service
```

Flow:

1. Webhook Service verifies provider signature and deduplicates delivery.
2. Repository Service normalizes repository, branch, commit, pull request, and developer metadata.
3. Repository Service publishes source events.
4. Pipeline Service consumes source events, evaluates triggers, and creates pipeline runs.
5. Pipeline workers execute asynchronous run steps.

## Pipeline to Build Interaction

```text
Pipeline Service -> Build Service -> Pipeline Runner Workers -> Build Service -> Event Bus
```

Flow:

1. Pipeline Service starts a run and requests build execution.
2. Build Service records build run metadata and execution intent.
3. Workers execute build tasks and report status.
4. Build Service records artifacts, provenance, logs, and produced images.
5. Build Service publishes build and artifact events.

## Build to Registry Interaction

```text
Build Service -> Container Registry Service -> Security/Scan Workers -> Event Bus
```

Flow:

1. Build Service reports produced image references and digests.
2. Container Registry Service verifies registry metadata and image identity.
3. Scan workers attach SBOM, signature, vulnerability, and provenance summaries.
4. Image events update deployment eligibility and topology graph edges.

## Deployment Interaction

```text
Deployment Service -> Kubernetes Service -> Kubernetes API
Deployment Service -> Monitoring Service -> Health Gates
Deployment Service -> Audit Service/Event Bus
```

Flow:

1. Deployment Service validates approval, policy, image eligibility, target environment, and idempotency.
2. Deployment Service records deployment operation state.
3. Kubernetes Service applies or coordinates runtime changes through controlled cluster identities.
4. Monitoring Service evaluates health gates.
5. Deployment Service records rollout results and publishes deployment events.

## Runtime Inventory Interaction

```text
Kubernetes Service -> Cluster Watches/Reconciliation -> Runtime Inventory -> Event Bus -> Topology Projection
```

Flow:

1. Kubernetes Service watches or polls cluster resources.
2. Runtime inventory records namespaces, nodes, pods, containers, services, and ingresses.
3. Runtime events publish observed topology edges.
4. Topology workers merge runtime edges with delivery lineage.
5. API topology reads use domain records and derived graph projections.

## Observability Interaction

```text
Monitoring/Logging Integrations -> Monitoring Service/Logging Service -> Alerts/Topology/AI
```

Flow:

1. Monitoring and Logging Services ingest metadata, pointers, and summaries.
2. Signals are correlated to deployment, application, cluster, pod, container, trace, and log context.
3. Alerts are raised and published.
4. Notifications and AI recommendations consume alert and health events.
5. Topology graph overlays health and signal summaries onto nodes.

## Notification Interaction

```text
Domain Event -> Notification Service -> Provider Integration -> User/Team Channel
```

Flow:

1. Domain service publishes user-visible or action-required event.
2. Notification Service evaluates preferences, routing, deduplication, and escalation.
3. Integration Service provides provider metadata and credential references.
4. Notification Service dispatches and records delivery state.

## AI Interaction

```text
User/Automation -> AI Service -> Authorized Context Reads -> Recommendation/Tool Proposal -> Audit/Event Bus
```

Flow:

1. AI Service authenticates and authorizes the caller.
2. AI Service retrieves only permission-filtered context from domain APIs, read models, logs, metrics, and topology projections.
3. AI Service generates explanations, summaries, or recommendations.
4. Tool execution proposals are policy-checked and approval-gated where required.
5. Audit Service records context access, tool proposals, approvals, executions, and outcomes.

## Billing Interaction

```text
Usage Events -> Billing Service -> Entitlements/Subscriptions -> Organization/Project Limits
```

Flow:

1. Services publish usage facts.
2. Billing Service aggregates usage by organization and billing period.
3. Subscription state updates entitlements and quotas.
4. Organization, Project, Pipeline, AI, and other services consume entitlement changes.

## Webhook Interaction

Webhook Service is the only generic inbound provider-event edge. Domain services should not each implement unrelated webhook verification logic.

Flow:

1. Provider sends webhook.
2. Webhook Service authenticates provider payload.
3. Webhook Service deduplicates and normalizes event metadata.
4. Domain-specific event is routed to Repository, Registry, Billing, Integration, or other owning service.
5. Raw payload handling follows retention and sensitivity policy.

## Scheduler Interaction

Scheduler Service emits time-based triggers to the event bus or calls internal orchestration APIs when immediate consistency is required.

Examples:

- Scheduled pipeline triggers.
- Periodic topology refresh.
- Credential rotation reminders.
- Billing aggregation.
- Retention and archival jobs.
- Drift detection.
- Health check sweeps.

## Cache Interaction

Redis is shared infrastructure but not shared business ownership.

Rules:

- Services own their cache key namespaces.
- Tenant data cache keys include organization scope.
- Cache invalidation follows domain events.
- Cache misses must fall back to authoritative storage or controlled dependency calls.
- Cache failure must not create security bypasses or corrupt state.

## File Storage Interaction

Large files are stored outside PostgreSQL. Services coordinate metadata and access.

Flow:

1. Owning service creates file metadata and upload/download intent.
2. File storage returns short-lived access target.
3. Client or worker transfers content.
4. Owning service finalizes checksum, scan state, retention policy, and references.
5. Audit records are emitted for sensitive access and exports.

## Topology Graph Collaboration

The topology graph is built from domain-owned facts:

```text
Developer -> Repository -> Pipeline -> Build -> Image -> Registry -> Deployment -> Cluster -> Node -> Pod -> Service -> Ingress -> Domain
```

Service collaboration:

- User Service identifies developers and actors.
- Repository Service links developers to repositories, branches, commits, and pull requests.
- Pipeline Service links repositories and branches to pipeline runs.
- Build Service links pipeline runs to builds, artifacts, and produced image references.
- Container Registry Service links images to registries, digests, tags, SBOMs, signatures, and scans.
- Deployment Service links builds/images to deployments, applications, environments, and clusters.
- Kubernetes Service links deployments to runtime inventory including nodes, pods, containers, services, ingresses, and domains.
- Monitoring and Logging Services attach health, metrics, logs, and alerts to graph nodes.
- Audit Service attaches actors, approvals, and privileged actions to graph edges.
- AI Service consumes authorized graph views for explanation and recommendation.

Real-time update model:

1. Domain services publish lineage and runtime events as changes occur.
2. Topology projection workers consume events and update current graph projections.
3. Reconciliation workers periodically verify graph state against authoritative services.
4. WebSocket subscriptions notify clients of topology changes.
5. REST topology APIs serve current, scoped, and historical graph views with freshness metadata.

## Failure Mode Interactions

- If event bus is delayed, synchronous commands still complete only after authoritative state is recorded; projections may show stale freshness.
- If topology projection fails, it is rebuilt from domain records and events.
- If external providers fail, circuit breakers prevent cascading failures and operations enter degraded states.
- If Redis is unavailable, services fall back to authoritative storage where safe and reject requests where rate/security state cannot be guaranteed.
- If AI providers fail, AI features degrade without affecting core delivery workflows.
