# SkyOps Database Principles

## Purpose

This document defines the database philosophy, guardrails, and operating principles for the SkyOps PostgreSQL architecture. It is intentionally a design document only. It does not define SQL, migrations, backend code, APIs, or UI.

## Database Philosophy

SkyOps uses PostgreSQL as the primary system of record for enterprise SaaS data because PostgreSQL provides strong relational integrity, transactional correctness, mature indexing, partitioning, JSON support for controlled metadata, row-level security capabilities, and operational maturity at scale.

The database architecture follows these principles:

1. **Tenant-aware by default:** Tenant ownership must be present on every tenant-scoped entity and every query path that touches customer data.
2. **Relational truth, derived projections:** PostgreSQL stores authoritative business state. Search indexes, caches, warehouses, graph projections, metrics stores, and vector indexes are derived systems.
3. **Explicit relationships:** Every topology, audit, security, and delivery relationship must be queryable through foreign keys, association entities, or immutable lineage records.
4. **Domain ownership:** Tables are owned by product domains aligned to SkyOps services. Domains may reference stable identifiers from other domains but must not mutate another domain's internal state.
5. **Append where history matters:** Audit logs, pipeline runs, build records, deployment events, metric samples, log envelopes, AI conversations, and billing ledger data must preserve history rather than overwrite operational facts.
6. **Soft delete for business entities:** User-visible business entities use soft delete to preserve auditability and relationship reconstruction. High-volume telemetry uses retention and partition expiration instead.
7. **Security and compliance first:** Sensitive data is encrypted, minimized, isolated, access-controlled, audited, and retained only as long as required.
8. **Scale through partitioning and lifecycle management:** Large tables are partitioned by tenant, time, or both. Cold data is archived before it harms operational performance.
9. **Schema discipline:** Every schema change requires review, compatibility planning, backfill strategy, rollback strategy, and observability.
10. **No hidden coupling:** Services do not reach through tables owned by other domains for write operations. Shared read models must be documented.

## Multi-Tenant Strategy

SkyOps is a multi-tenant SaaS platform supporting thousands of organizations and millions of users. The default tenancy model is shared PostgreSQL clusters with strict logical isolation. Dedicated clusters or databases may be offered for large regulated enterprise tenants without changing the logical data model.

### Tenant Hierarchy

The canonical tenant hierarchy is:

```text
Organization -> Workspace -> Project -> Environment -> Runtime Resources
```

The minimum required tenant key for customer-owned rows is `organization_id`. Entities that are naturally scoped deeper should also include `workspace_id`, `project_id`, and `environment_id` as appropriate.

### Isolation Requirements

- Every tenant-scoped table must include `organization_id` unless it is a global catalog table or join table whose parents enforce tenant scope.
- Query paths must include tenant predicates and authorization context.
- Unique constraints for tenant-owned names must be scoped by organization or by the appropriate parent entity.
- Caches, queues, events, logs, AI context, and derived projections must carry tenant identifiers.
- Cross-tenant administration tables must be separately permissioned and audited.

### Enterprise Tenant Options

The physical deployment strategy supports three tiers:

1. **Shared cluster, shared database:** Default SaaS tier. Logical tenant isolation with tenant-scoped constraints, policies, and indexes.
2. **Shared cluster, dedicated database:** Enterprise isolation tier for large customers requiring stronger blast-radius control.
3. **Dedicated cluster:** Regulated or hyperscale tenant tier with independent backup, restore, encryption, and maintenance windows.

The logical schema remains consistent across tiers to preserve product behavior and operational tooling.

## Identifier Strategy

- Use globally unique, non-sequential public identifiers for entities exposed externally.
- Internal database identifiers must be stable and immutable.
- External provider identifiers must be stored separately from SkyOps identifiers.
- Avoid encoding business meaning in identifiers.
- Relationship tables must use explicit references rather than relying on names or external IDs.

## Time and History Strategy

- Store all timestamps in UTC.
- Track `created_at`, `updated_at`, and `deleted_at` on soft-deletable business entities.
- Track `created_by`, `updated_by`, and `deleted_by` where user attribution matters.
- Immutable event-like entities should store event time, ingestion time, source, actor, and correlation IDs.
- Do not overwrite historical execution facts to match current names or configuration.

## Soft Delete Strategy

Business entities use soft delete to preserve auditability and topology reconstruction. Soft-deleted records remain queryable by privileged audit and compliance workflows but are hidden from normal product queries.

Soft delete rules:

- Use `deleted_at` and `deleted_by` semantics in design.
- Child entities must define whether they are cascaded, retained, detached, or archived when a parent is deleted.
- Names may be reusable after deletion only when uniqueness policies explicitly allow it.
- Hard deletion is reserved for legal erasure workflows, retention expiry, ephemeral telemetry, and operational partitions approved for removal.

## Auditability Strategy

Every user, service, AI, or system action that changes security posture, delivery state, infrastructure state, billing, tenant configuration, or production runtime state must produce an audit record.

Audit records must capture:

- Organization and tenant scope.
- Actor identity and actor type.
- Target entity type and identifier.
- Action, result, reason, and correlation ID.
- Before and after summaries for sensitive mutations where safe.
- Source IP, user agent, integration, or service identity where applicable.
- Immutable timestamp.

## Topology Graph Strategy

SkyOps must reconstruct the application topology graph from developer action to runtime exposure. The relational model stores direct edges through foreign keys and association entities. A derived graph projection may be built from these relationships for fast graph traversal, but PostgreSQL remains the source of truth.

Required canonical path:

```text
Developer -> Organization -> Project -> Repository -> Branch -> Pipeline -> Build -> Container Image -> Registry -> Deployment -> Cluster -> Node -> Pod -> Service -> Ingress -> Domain -> Application
```

Design rules:

- Every edge in the canonical path must be represented by a foreign key or association entity.
- Historical runs must preserve lineage even if current repositories, branches, clusters, or services are renamed or deleted.
- Runtime Kubernetes inventory must include provider UIDs, cluster scope, namespace scope, ownership references, and discovery timestamps.
- The topology model must support both current-state queries and point-in-time reconstruction.

## Partitioning Strategy

Partition high-volume and append-heavy tables before they become operational risks.

Recommended partitioning:

- Audit logs: by time, optionally subpartitioned or indexed by organization.
- Pipeline runs, build runs, deployment events: by time and organization-aware indexes.
- Metrics samples: by time with short hot retention in PostgreSQL; long-term metrics belong in a metrics backend.
- Log envelopes: by time with short retention or metadata-only records in PostgreSQL; full log bodies belong in log storage.
- Notifications: by organization and time for large tenants.
- AI messages and recommendation events: by organization and time.
- Sessions and API key usage: by time.
- Billing usage records: by billing period and organization.

Partition keys must preserve common query patterns. Partition maintenance must be automated, observable, and tested.

## Indexing Strategy

Indexes must serve documented query patterns. Every index has write cost and storage cost.

Baseline index rules:

- Index every foreign key used in joins.
- Index tenant predicates used in application queries.
- Use composite indexes that begin with tenant scope for tenant-owned entities.
- Use partial indexes for active records when soft delete is common.
- Use time-based indexes for event and telemetry tables.
- Use unique indexes scoped to parent ownership, such as organization plus slug.
- Avoid over-indexing low-selectivity boolean columns unless used in partial indexes.
- Review slow query logs and query plans before adding speculative indexes.

## Backup and Restore Strategy

PostgreSQL backup strategy must support enterprise recovery objectives.

- Use continuous write-ahead-log archiving for point-in-time recovery.
- Take regular full backups and verify them through automated restore tests.
- Encrypt backups at rest and in transit.
- Store backups in a separate security boundary from the primary database.
- Define RPO and RTO per environment and tenant tier.
- Support tenant-aware export and restore workflows for enterprise support, while recognizing that shared-database tenant restore requires careful reconciliation.
- Test disaster recovery procedures regularly.

## Data Retention Strategy

Retention must balance customer value, compliance, cost, and performance.

- Business records are retained while the customer account is active unless deleted under policy.
- Audit logs are retained according to enterprise compliance requirements and plan tier.
- High-volume telemetry has short hot retention in PostgreSQL and longer retention in specialized storage.
- AI conversation and prompt data must respect customer-configurable retention and privacy policy.
- Sessions, temporary tokens, webhook deliveries, and transient operational data expire aggressively.
- Billing records and invoices follow financial retention requirements.

## Archival Strategy

Cold data must move out of hot operational tables before it harms latency or maintenance.

- Archive immutable historical data to low-cost object storage or analytical stores with integrity manifests.
- Keep searchable metadata in PostgreSQL when required for product navigation.
- Preserve referential context needed for audits and topology reconstruction.
- Archives must be encrypted, access-controlled, and lifecycle-managed.
- Restore-from-archive workflows must be documented for support and compliance teams.

## Schema Change Strategy

Schema evolution must be backward compatible whenever possible.

- Use expand-and-contract migrations for production changes.
- Backfills must be resumable, idempotent, rate-limited, and observable.
- Large table changes require rollout plans and performance review.
- Avoid long blocking locks on production tables.
- Contract-breaking changes require API, event, and service compatibility planning.

## Data Classification

SkyOps data must be classified so storage, access, logs, and retention can be enforced consistently.

- **Public metadata:** Non-sensitive product metadata safe for broad internal display.
- **Tenant confidential:** Customer repositories, deployments, topology, infrastructure metadata, logs, metrics, and AI context.
- **Security sensitive:** Secrets references, tokens, API key hashes, session identifiers, vulnerability data, auth events, and audit logs.
- **Regulated financial:** Billing, subscription, invoice, and payment-provider metadata.
- **Operational telemetry:** Metrics, logs, traces, alerts, and runtime inventory.

## Encryption and Secret Storage

- Encrypt data at rest using managed storage encryption.
- Use column-level or application-level encryption for highly sensitive values when needed.
- Store secret values outside PostgreSQL in approved secret managers whenever possible.
- If secret metadata is stored in PostgreSQL, store references, versions, fingerprints, and policy metadata, not plaintext secret values.
- API keys and tokens must be stored as hashes or verifiable digests, never plaintext.

## Read Models and Analytics

Operational PostgreSQL tables should not be overloaded with expensive analytics. Use read replicas, materialized projections, event streams, warehouses, graph projections, and search systems for heavy read workloads. Derived systems must be rebuildable from authoritative relational data and events.
