# SkyOps PostgreSQL Database Design

## Purpose

This document defines the complete enterprise PostgreSQL database architecture for SkyOps. It is a design document only and intentionally contains no SQL, migrations, backend code, API definitions, or UI design.

This design supports the approved SkyOps Product Vision, Software Architecture, Monorepo Structure, and Engineering Standards. It focuses on multi-tenant enterprise SaaS scale, PostgreSQL correctness, platform topology, auditability, security, and long-term operational maintainability.

## Document Set

The SkyOps database architecture is split across three documents:

- `database-principles.md` defines philosophy, tenancy, partitioning, indexing, backup, retention, archival, and schema evolution principles.
- `entity-relationship.md` defines entity relationships, diagrams, and the required per-entity design metadata.
- `database-design.md` defines the end-to-end architecture and explains how the model operates as a system.

## Architectural Overview

SkyOps uses PostgreSQL as the authoritative transactional database for tenant, identity, access, delivery, Kubernetes inventory, application topology, integration, AI, notification, audit, billing, subscription, feature flag, and configuration metadata.

Specialized systems may support high-volume or domain-specific workloads:

- Object storage for archived logs, build artifacts, SBOM files, and export bundles.
- Metrics backends for long-term time-series analytics.
- Log storage/search systems for full log bodies.
- Search indexes for user-facing search.
- Vector stores for AI retrieval indexes.
- Graph projections for fast topology traversal.

These systems are derived from PostgreSQL records and event streams. PostgreSQL remains the source of truth for business relationships and topology lineage.

## Logical Domain Schemas

The conceptual PostgreSQL organization uses domain-oriented schemas or ownership groups:

1. **identity:** users, sessions, API keys, roles, permissions, teams, memberships.
2. **tenancy:** organizations, projects, environments, settings, feature flags.
3. **source:** repositories and branches.
4. **delivery:** pipelines, builds, deployments, release lineage.
5. **artifacts:** registries and images.
6. **kubernetes:** clusters, namespaces, nodes, pods, containers, services, ingresses, config maps, and secret metadata.
7. **application_catalog:** applications and domains.
8. **observability:** monitoring, metrics, logs, alerts.
9. **ai:** conversations and recommendations.
10. **integrations:** provider connections and webhook metadata.
11. **notifications:** notification events and delivery state.
12. **audit:** immutable audit records.
13. **billing:** billing accounts, subscriptions, usage rollups, invoices through provider references.

The exact physical schema names may evolve through ADRs, but ownership boundaries must remain clear.

## Multi-Tenant Architecture

The primary tenant boundary is `Organization`. Projects, environments, repositories, pipelines, builds, deployments, clusters, and runtime resources all carry organization scope directly or through their parent relationships.

### Tenant Query Model

Every tenant-scoped query must start from an authenticated principal and organization context. Application services must apply authorization and tenant predicates before reading or mutating data.

Recommended query shape:

```text
authenticated actor -> organization membership -> authorization policy -> tenant-scoped entity query
```

No product path may rely on client-side filtering or UI visibility as tenant isolation.

### Tenant Scale Model

For thousands of organizations and millions of users, the architecture supports:

- Composite indexes beginning with `organization_id` for tenant-owned entities.
- Time partitioning for high-volume append-only records.
- Optional tenant placement metadata for dedicated database or dedicated cluster enterprise tiers.
- Read replicas for reporting and large customer dashboards.
- Derived read models for graph, search, and analytics.
- Background archival to keep hot tables small.

## Core Entity Groups

### Identity and Access

The identity model includes Organization, User, Team, Role, Permission, API Key, and Session entities. It supports human users, teams, RBAC, service-like credentials, session management, and audit-safe identity history.

Design decisions:

- Authorization assignments are explicit through role-permission and user/team role mappings.
- API keys store only hashed secret material and scoped permissions.
- Sessions are revocable and expire aggressively.
- User deletion preserves audit actor references through soft deletion or anonymized immutable actor summaries.

### Product and Delivery

The product delivery model includes Project, Repository, Branch, Pipeline, Build, Image, Registry, Deployment, Environment, Application, and Domain entities.

Design decisions:

- Builds and deployments are immutable operational history.
- Repositories and branches are soft-deletable to retain delivery lineage.
- Images are referenced by digest to preserve supply-chain identity.
- Deployments connect build artifacts to environments, clusters, applications, and runtime state.

### Kubernetes Runtime Inventory

The runtime model includes Cluster, Namespace, Node, Pod, Container, Service, Ingress, Secret metadata, and ConfigMap metadata.

Design decisions:

- Kubernetes provider UIDs and names are stored for reconciliation and point-in-time mapping.
- Runtime observations carry timestamps so SkyOps can distinguish current state from historical state.
- Secret values are not stored in PostgreSQL; only references and metadata are stored.
- Services and ingresses create explicit topology edges from runtime workloads to external exposure.

### Observability and Operations

The operations model includes Monitoring, Metric, Log, Alert, Notification, Integration, and Audit Log entities.

Design decisions:

- PostgreSQL stores observability configuration, metadata, correlation records, and short-retention samples where needed.
- Full logs and long-term metrics should live in specialized stores and be referenced from PostgreSQL.
- Alerts correlate runtime signals to deployments, applications, and projects.
- Audit logs are immutable, partitioned, and retained according to compliance policy.

### AI and Platform Intelligence

The AI model includes AI Conversation and AI Recommendation entities.

Design decisions:

- AI context is tenant-scoped and permission-aware.
- Recommendations are auditable and track approval/execution outcomes.
- AI records reference concrete SkyOps entities so recommendations can be explained and reviewed.
- AI retention may be configured per customer policy.

### Commercial and Configuration

The commercial and platform configuration model includes Billing, Subscription, Feature Flag, and System Settings entities.

Design decisions:

- Billing and subscription history is retained for financial compliance.
- Feature flags can be global, organization-scoped, project-scoped, or environment-scoped.
- System settings must avoid plaintext secrets and must audit all changes.

## Application Topology Graph Support

SkyOps must answer:

1. What is running in production?
2. How did it get there?
3. What should we do next?

The topology graph is supported by relational edges across identity, delivery, artifact, deployment, Kubernetes, ingress, domain, and application entities.

### Canonical Graph Path

```text
Developer
  -> Organization
  -> Project
  -> Repository
  -> Branch
  -> Pipeline
  -> Build
  -> Container Image
  -> Registry
  -> Deployment
  -> Cluster
  -> Node
  -> Pod
  -> Service
  -> Ingress
  -> Domain
  -> Application
```

### Graph Design Decisions

- Developer is represented by User plus commit/build/deployment actor records.
- Project is the product ownership boundary for repositories, environments, applications, and deployments.
- Repository and Branch connect source events to pipelines and builds.
- Build records preserve commit SHA, actor, pipeline, produced image, provenance, and artifact metadata.
- Image records connect build output to registry storage and runtime containers by digest.
- Deployment records connect build/image lineage to environment, cluster, and application.
- Kubernetes inventory connects deployment to namespace, pod, container, service, ingress, and domain exposure.
- Application provides the stable product-level node that links source, runtime, observability, alerts, and external domains.

### Current-State and Point-in-Time Queries

Current-state topology uses active application, deployment, cluster, namespace, pod, service, ingress, and domain records.

Point-in-time topology uses:

- Build start/end timestamps and immutable commit data.
- Deployment start/end timestamps, status transitions, and rollback links.
- Runtime resource observation windows.
- Image digests and registry metadata.
- Audit logs and event records for actor and approval context.

## Scalability Decisions

### Write Scalability

- Append-heavy tables are partitioned by time.
- Tenant-scoped operational records include organization-aware indexes.
- High-volume telemetry is stored in specialized systems, with PostgreSQL retaining metadata and correlations.
- Background jobs perform batch updates and archival outside request paths.
- Large tenants can be moved to dedicated database or cluster tiers using organization placement metadata.

### Read Scalability

- Common product queries use composite indexes aligned to tenant, parent, status, and time filters.
- Read replicas support dashboards, reporting, support tools, and internal analytics.
- Materialized read models may summarize deployment health, application topology, billing usage, and alert state.
- Graph projections may be built for low-latency topology traversal while remaining rebuildable from source tables.

### Operational Scalability

- Partition lifecycle management is automated.
- Vacuum, analyze, and index maintenance are monitored.
- Slow queries are tracked and reviewed.
- Schema changes use expand-and-contract rollout patterns.
- Backfills are resumable, rate-limited, and tenant-aware.

## Partitioning Strategy

Partitioning is mandatory for tables expected to grow continuously.

### Time-Partitioned Tables

- Audit Log.
- Build run history at large scale.
- Deployment events and status transitions.
- Metrics samples retained in PostgreSQL.
- Log envelopes retained in PostgreSQL.
- Alert occurrences.
- Notification deliveries.
- AI conversation messages and recommendation events when volume requires it.
- API key usage and session security events.
- Billing usage records.

### Tenant-Aware Access Within Partitions

Most partitions remain time-based for operational simplicity, with organization-aware composite indexes inside partitions. For hyperscale tenants, dedicated physical placement avoids one tenant dominating shared partitions.

### Partition Maintenance

Partition creation, retention expiration, archival export, and integrity checks must be automated. Partition operations must be observable and must not block normal application traffic.

## Indexing Strategy

The baseline indexing strategy is:

- Primary identifier index for every entity.
- Foreign key indexes for every join path.
- Tenant-parent composite indexes such as organization plus project, project plus repository, repository plus branch, pipeline plus build, and application plus deployment.
- Status/time indexes for operational lists.
- Partial active-record indexes for soft-deletable tables.
- Unique scoped indexes for slugs and external provider identifiers.
- Digest indexes for images and API keys.
- Correlation ID, trace ID, and request ID indexes for diagnostics tables.

Indexes must be justified by query patterns and reviewed during performance tuning.

## Data Retention Strategy

Retention is domain-specific:

- Organizations, projects, repositories, environments, applications, and core configuration are retained while active and soft-deleted after removal.
- Build and deployment history is retained according to plan and compliance tier, then archived.
- Audit logs follow enterprise compliance retention and are archived before deletion.
- Metrics and log samples have short hot retention in PostgreSQL; long-term detail belongs in observability storage.
- AI conversation data follows customer-configurable retention and privacy requirements.
- Sessions, transient tokens, webhook deliveries, and notification attempts expire aggressively.
- Billing and subscription records follow financial and legal retention requirements.

## Archival Strategy

Archive candidates include old build history, old deployment events, old audit partitions, old alert occurrences, expired notifications, AI conversations beyond active retention, and telemetry envelopes.

Archive design requirements:

- Export immutable records with checksums and manifests.
- Preserve tenant, entity, and time range metadata.
- Encrypt archives and restrict access.
- Keep minimal searchable metadata in PostgreSQL where product workflows require it.
- Provide documented restore procedures for compliance and support.

## Backup Strategy

SkyOps PostgreSQL backup architecture must include:

- Continuous point-in-time recovery through WAL archiving.
- Scheduled full backups.
- Cross-region backup copies for production.
- Encryption at rest and in transit.
- Automated restore testing.
- Environment-specific RPO/RTO definitions.
- Tenant-aware export support for enterprise customers.
- Runbooks for full restore, point-in-time restore, accidental deletion recovery, and regional disaster recovery.

## Security and Compliance Decisions

- Store secret references, not plaintext secret values.
- Store API keys and session tokens as hashes or digests.
- Classify data by sensitivity and enforce access accordingly.
- Audit privileged reads and all sensitive writes.
- Apply least privilege to application database roles.
- Separate migration roles from runtime roles.
- Prevent cross-tenant access through application authorization and database-level guardrails where appropriate.
- Minimize sensitive data replicated to analytics, AI, logs, and support tooling.

## Availability and Disaster Recovery

Production PostgreSQL must run with high availability appropriate for enterprise SaaS:

- Multi-zone primary/replica topology.
- Automated failover with tested procedures.
- Read replicas for heavy read paths.
- Connection pooling for service scalability.
- Capacity monitoring for storage, CPU, memory, replication lag, locks, and bloat.
- Disaster recovery drills with measured RTO and RPO.

## Data Quality and Integrity

- Use foreign keys for authoritative relationships where write throughput and lifecycle permit.
- Use typed target references only for polymorphic audit and event records where direct foreign keys would create excessive coupling.
- Use immutable records for historical facts.
- Validate external provider IDs and normalize names consistently.
- Keep timestamps, actor references, and correlation IDs on operational records.
- Reconciliation workers must detect drift between external systems and SkyOps records.

## Reporting and Analytics

Operational PostgreSQL is not the long-term analytics warehouse. Reporting should use read replicas or derived analytical stores. Metrics such as deployment frequency, lead time, change failure rate, recovery time, platform adoption, AI recommendation outcomes, and billing usage should be computed from immutable delivery, deployment, audit, and usage records.

## Design Boundaries

This database design intentionally does not define:

- SQL table definitions.
- Column types.
- Migrations.
- Backend repositories or services.
- API endpoints.
- UI workflows.
- Provider-specific implementation code.

Those implementation details must be introduced later through approved architecture, ADRs, engineering standards, and code review.
