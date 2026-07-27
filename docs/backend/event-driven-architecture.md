# SkyOps Event-Driven Backend Architecture

## Purpose

This document defines the event-driven architecture for SkyOps backend services. It is a design document only and contains no implementation code, event handler code, or database migrations.

## Event Philosophy

Events are durable facts about meaningful changes in SkyOps. Events enable loose coupling, asynchronous workflows, topology graph updates, audit enrichment, notifications, AI context building, billing aggregation, and integrations.

Event rules:

- Publish events after authoritative state changes are accepted.
- Events are immutable.
- Events are versioned contracts.
- Consumers are idempotent.
- Events contain tenant and correlation context.
- Sensitive payloads are minimized and classified.
- Event schemas are owned by the producing domain.

## Event Bus Responsibilities

The event bus provides:

- Durable publish/subscribe delivery.
- Subject or topic routing by domain and event type.
- Consumer groups for horizontally scalable workers.
- Replay where retention permits.
- Dead-letter handling for poison messages.
- Backpressure and delivery observability.
- Schema version visibility.

The event bus is not a database replacement. Authoritative state remains in domain services and PostgreSQL.

## Event Envelope Standard

Every event conceptually includes:

- Event ID.
- Event type.
- Schema version.
- Organization ID.
- Workspace/project/environment scope where applicable.
- Actor ID and actor type where applicable.
- Source service.
- Resource type and resource ID.
- Correlation ID.
- Causation ID.
- Occurred timestamp.
- Published timestamp.
- Payload classification.
- Data payload.

## Event Naming Strategy

Event names should be stable, domain-scoped, and past tense.

Examples:

- `repository.connected`
- `branch.observed`
- `pipeline.run.started`
- `build.completed`
- `image.produced`
- `deployment.started`
- `cluster.connected`
- `pod.observed`
- `ingress.observed`
- `alert.raised`
- `ai.recommendation.created`
- `audit.recorded`
- `subscription.changed`

## Event Categories

### Domain Events

Domain events describe business state changes owned by a service. Examples include organization created, project archived, pipeline run completed, deployment failed, and integration disconnected.

### Operational Events

Operational events describe runtime or infrastructure observations such as pod observed, node not ready, metric threshold breached, and log envelope indexed.

### Audit Events

Audit events describe security-sensitive actions and are normalized by the Audit Service into immutable evidence records.

### Workflow Events

Workflow events trigger asynchronous processes such as scans, notifications, AI context updates, topology projection updates, and billing aggregation.

### Integration Events

Integration events represent normalized external provider payloads after webhook verification and provider-specific parsing.

## Publishing Strategy

Producers must publish events consistently with state changes. The preferred pattern is to persist authoritative state and event publication intent atomically through a transactional outbox or equivalent reliability mechanism.

Publishing requirements:

- Do not publish events for failed or rolled-back state changes.
- Include enough identifiers for consumers to fetch additional data from owning services.
- Do not include secrets or large payloads.
- Include schema version and payload classification.
- Use causation IDs to connect workflows.

## Consumption Strategy

Consumers must be resilient to duplicates, delays, reordering, and schema evolution.

Consumer requirements:

- Idempotency by event ID and domain key.
- Safe retries with backoff.
- Dead-letter handling.
- Metrics for lag, failure rate, retry count, and processing duration.
- Compatibility with older event versions during rollout.
- Authorization and tenant checks before side effects where required.

## Retry and Dead-Letter Strategy

Event processing retries must be bounded and observable.

Rules:

- Transient failures retry with exponential backoff and jitter.
- Permanent validation or schema failures move to dead-letter workflows.
- Poison events must not block an entire partition indefinitely.
- Dead-letter events require owner alerting and remediation playbooks.
- Reprocessing must be idempotent.

## Event Security

- Events carry tenant scope.
- Sensitive data is minimized.
- Secrets are never published.
- Event consumers are authorized by service identity.
- Cross-tenant event consumption is restricted to approved platform services.
- Event payload access is logged or audited when sensitive.
- AI consumers receive only policy-approved event fields.

## Event Schema Evolution

Event schemas evolve compatibly by default.

Rules:

- Additive optional fields are preferred.
- Removing fields, changing semantics, or changing required fields requires a new schema version.
- Producers and consumers must support rolling deploys.
- Event schema changes require contract review.
- Deprecated event versions require usage monitoring and sunset plans.

## Topology Event Architecture

The Application Topology Graph is updated through domain events and runtime observation events.

Canonical graph path:

```text
Developer -> Repository -> Pipeline -> Build -> Image -> Registry -> Deployment -> Cluster -> Node -> Pod -> Service -> Ingress -> Domain
```

Topology event sources:

- User Service publishes actor identity and membership changes.
- Repository Service publishes repository, branch, commit, pull request, and developer contribution events.
- Pipeline Service publishes pipeline definition and run events.
- Build Service publishes build and artifact events.
- Container Registry Service publishes image, registry, SBOM, signature, and scan events.
- Deployment Service publishes deployment lifecycle and environment/cluster target events.
- Kubernetes Service publishes cluster, namespace, node, pod, container, service, ingress, and domain observation events.
- Monitoring and Logging Services publish health, alert, metric, log, and trace correlation events.
- Audit Service publishes audit-record availability events for sensitive graph edges.

Topology projection workers consume these events to update derived graph views. The graph projection stores current edges, historical edges, freshness metadata, source event IDs, and confidence/observation status. If projections are corrupted or stale, they are rebuilt from domain records and retained events.

## Event-Driven Workflow Examples

### Repository Push to Pipeline Run

1. Webhook Service receives verified Git provider event.
2. Repository Service normalizes commit and branch facts.
3. Repository Service publishes source events.
4. Pipeline Service consumes source events and evaluates triggers.
5. Pipeline Service publishes pipeline run events.
6. Build workers begin execution.

### Build to Deployment Eligibility

1. Build Service publishes build completed and image produced events.
2. Container Registry Service verifies image metadata and scan status.
3. Security and policy workers evaluate eligibility.
4. Deployment Service consumes eligibility events before promotion.

### Deployment to Runtime Topology

1. Deployment Service publishes deployment started and completed events.
2. Kubernetes Service observes runtime resources created or updated by the deployment.
3. Kubernetes Service publishes runtime topology events.
4. Topology workers connect deployment to pods, containers, services, ingresses, and domains.
5. Monitoring Service overlays health state.

### Alert to AI Recommendation

1. Monitoring Service publishes alert raised event.
2. Notification Service dispatches alerts to configured recipients.
3. AI Service consumes alert and topology context events.
4. AI Service creates a permission-filtered recommendation.
5. Audit Service records AI recommendation and any approved action.

## Event Retention and Replay

Retention depends on event category:

- Security and audit-related events require long retention or audit normalization.
- Topology and delivery events require enough retention to rebuild projections and support historical views.
- High-volume telemetry events may have short event-bus retention after summarization.
- Billing usage events retain according to financial policy.

Replay must be controlled, tenant-aware, and idempotent. Replays should not resend customer notifications or repeat external side effects unless explicitly requested.

## Event Observability

Required event observability:

- Publish success/failure metrics.
- Consumer lag metrics.
- Dead-letter counts.
- Processing latency.
- Retry counts.
- Event throughput by domain and tenant tier.
- Schema version adoption.
- Topology projection freshness.

## Event Ownership

Each domain owns the events it publishes. Ownership includes schema documentation, compatibility guarantees, examples, deprecation plans, and support for consumers during migration.
