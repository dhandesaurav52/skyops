# SkyOps Backend Service Catalog

## Purpose

This catalog defines the official backend services for SkyOps and documents each service's purpose, responsibilities, interfaces, dependencies, events, storage, cache usage, security considerations, and future expansion. It is a design document only and contains no implementation code.

## Service Catalog Standards

Each service must:

- Own a bounded context.
- Publish events for meaningful state changes.
- Consume events through versioned contracts.
- Enforce tenant isolation and authorization.
- Expose health, metrics, traces, logs, and audit hooks.
- Avoid direct writes to another service's data.

## 1. API Gateway

- **Purpose:** Provides the secure edge entry point for SkyOps APIs and real-time connections.
- **Responsibilities:** TLS enforcement, routing, API version selection, authentication delegation, request/correlation IDs, rate limiting, request size limits, CORS, access logging, and gateway telemetry.
- **Public Interfaces:** Public REST entry point, WebSocket entry point, upload-session routing, health endpoints.
- **Internal Dependencies:** Authentication Service, rate limit store, service discovery, observability stack.
- **Events Published:** Gateway request anomaly events, rate limit violation events, authentication failure signals where appropriate.
- **Events Consumed:** Configuration change events, service route update events, rate limit policy events.
- **Storage Used:** No authoritative business storage; may use gateway configuration store.
- **Cache Usage:** Redis for rate counters, short-lived routing/policy cache, and abuse detection state.
- **Security Considerations:** Must not contain domain business logic; must enforce transport security and protect headers from spoofing.
- **Future Expansion:** Multi-region edge routing, tenant-aware edge policies, advanced bot protection, and API marketplace support.

## 2. Authentication Service

- **Purpose:** Authenticates users, service identities, sessions, API keys, and enterprise identity provider flows.
- **Responsibilities:** Login flows, token validation, session lifecycle, API key verification, SSO/OIDC/SAML integration, MFA coordination, credential revocation, and authentication risk signals.
- **Public Interfaces:** Authentication endpoints, session endpoints, token refresh/revocation endpoints, SSO callback surfaces.
- **Internal Dependencies:** User Service, Organization Service, Audit Service, Integration Service, secret manager, Redis.
- **Events Published:** User authenticated, authentication failed, session created, session revoked, API key authenticated, credential revoked.
- **Events Consumed:** User disabled, organization suspended, role changed, identity provider configuration changed.
- **Storage Used:** Session metadata, credential metadata, API key digests, identity provider bindings.
- **Cache Usage:** Redis for session acceleration, revocation checks, login throttling, and short-lived token metadata.
- **Security Considerations:** Never store plaintext API keys or tokens; enforce issuer/audience/expiry/revocation; audit sensitive authentication events.
- **Future Expansion:** Passwordless login, adaptive authentication, device trust, enterprise SCIM-triggered deprovisioning, and step-up authentication.

## 3. User Service

- **Purpose:** Manages human user profiles, memberships, team links, and user lifecycle metadata.
- **Responsibilities:** User profile lifecycle, organization memberships, team memberships, status changes, user preferences, identity profile synchronization, and user lookup for audit/topology actors.
- **Public Interfaces:** User profile and membership APIs for authorized clients.
- **Internal Dependencies:** Organization Service, Authentication Service, Audit Service, Notification Service.
- **Events Published:** User created, user updated, user disabled, user membership added, user membership removed, team membership changed.
- **Events Consumed:** Organization created, organization suspended, SSO identity updated, billing entitlement changed where user limits apply.
- **Storage Used:** PostgreSQL user and membership records.
- **Cache Usage:** Short-lived user profile and membership cache with event-driven invalidation.
- **Security Considerations:** Protect personal data, preserve audit actor references, enforce tenant-scoped user visibility.
- **Future Expansion:** User lifecycle analytics, profile federation, SCIM synchronization, and delegated administration.

## 4. Organization Service

- **Purpose:** Owns tenant lifecycle, organization settings, enterprise boundaries, and high-level governance metadata.
- **Responsibilities:** Organization creation, suspension, region/plan metadata, workspace hierarchy where applicable, tenant settings, organization ownership, and tenant placement metadata.
- **Public Interfaces:** Organization administration APIs and internal tenancy resolution APIs.
- **Internal Dependencies:** Billing Service, User Service, Audit Service, Feature/config store, secret manager for tenant-level references.
- **Events Published:** Organization created, organization updated, organization suspended, organization reactivated, tenant setting changed.
- **Events Consumed:** Billing status changed, subscription changed, enterprise contract changed.
- **Storage Used:** PostgreSQL organization and tenant settings records.
- **Cache Usage:** Tenant settings and placement cache with strict invalidation.
- **Security Considerations:** Organization changes are high-risk and require strong authorization and audit.
- **Future Expansion:** Dedicated tenant placement automation, enterprise hierarchy, regional data residency controls, and tenant export workflows.

## 5. Project Service

- **Purpose:** Owns project lifecycle and product ownership boundaries inside organizations.
- **Responsibilities:** Project CRUD lifecycle, ownership assignment, application grouping, environment associations, project-level policies, and topology root metadata.
- **Public Interfaces:** Project and application catalog APIs; internal project resolution APIs.
- **Internal Dependencies:** Organization Service, User Service, Audit Service, Billing Service for quotas, Feature Flag configuration.
- **Events Published:** Project created, project updated, project archived, project owner changed, application created, application updated.
- **Events Consumed:** Organization suspended, team changed, entitlement changed, deployment created for project summary updates.
- **Storage Used:** PostgreSQL project and application catalog records.
- **Cache Usage:** Project metadata cache, application summary cache, quota cache.
- **Security Considerations:** Enforce project-scoped permissions and avoid exposing project existence across tenants.
- **Future Expansion:** Project templates, ownership scoring, service catalog maturity metadata, and project-level cost allocation.

## 6. Repository Service

- **Purpose:** Manages source repository connections, branches, commits, pull request metadata, and source-to-project mapping.
- **Responsibilities:** Git provider integration metadata, repository sync, branch inventory, webhook normalization, commit/developer attribution, and repository topology edges.
- **Public Interfaces:** Repository, branch, and source metadata APIs.
- **Internal Dependencies:** Integration Service, Webhook Service, Project Service, User Service, Audit Service.
- **Events Published:** Repository connected, repository disconnected, branch observed, commit observed, pull request observed, repository mapped to project.
- **Events Consumed:** Git webhook received, integration disconnected, project archived, user identity linked.
- **Storage Used:** PostgreSQL repository, branch, commit metadata, and provider mapping records.
- **Cache Usage:** Repository lookup cache, provider installation cache, branch summary cache.
- **Security Considerations:** Provider tokens remain in secret manager; repository visibility is permission-filtered; webhook signatures must be validated upstream.
- **Future Expansion:** Code ownership analysis, repository health scoring, monorepo service detection, and dependency graph integration.

## 7. Pipeline Service

- **Purpose:** Owns CI/CD pipeline definitions, triggers, policies, and run orchestration metadata.
- **Responsibilities:** Pipeline definition lifecycle, trigger evaluation, run state coordination, approvals, retries, cancellation, and pipeline-to-repository topology edges.
- **Public Interfaces:** Pipeline definition and run APIs; internal orchestration APIs for workers.
- **Internal Dependencies:** Repository Service, Project Service, Build Service, Audit Service, Scheduler Service, Event Bus.
- **Events Published:** Pipeline created, pipeline updated, pipeline triggered, pipeline run started, pipeline run completed, pipeline run failed, approval required.
- **Events Consumed:** Commit observed, pull request updated, schedule fired, manual trigger requested, policy result received.
- **Storage Used:** PostgreSQL pipeline definitions and run metadata.
- **Cache Usage:** Trigger rule cache, pipeline definition cache, active run cache.
- **Security Considerations:** Enforce repository/project permissions, protect secrets, audit manual triggers and approvals.
- **Future Expansion:** Visual workflow graph, reusable pipeline templates, matrix execution, and policy-aware auto-remediation.

## 8. Build Service

- **Purpose:** Owns build execution records, artifact metadata, provenance, and build-to-image lineage.
- **Responsibilities:** Build run tracking, artifact registration, provenance capture, log/artifact references, status transitions, and handoff to registry/deployment domains.
- **Public Interfaces:** Build and artifact metadata APIs; internal worker callback APIs.
- **Internal Dependencies:** Pipeline Service, Repository Service, Container Registry Service, File Storage, Audit Service.
- **Events Published:** Build started, build completed, build failed, artifact produced, image produced, provenance recorded.
- **Events Consumed:** Pipeline run started, worker status updated, image scan completed, artifact uploaded.
- **Storage Used:** PostgreSQL build metadata; object storage for artifacts and logs.
- **Cache Usage:** Active build status cache and artifact lookup cache.
- **Security Considerations:** Preserve supply-chain integrity, protect build logs from secret leakage, audit artifact promotion.
- **Future Expansion:** SLSA provenance, build cache metadata, SBOM enrichment, and distributed build execution.

## 9. Deployment Service

- **Purpose:** Owns release and deployment lifecycle across environments and clusters.
- **Responsibilities:** Deployment planning, approvals, promotion, rollout tracking, rollback records, deployment health summaries, environment protections, and deployment topology edges.
- **Public Interfaces:** Deployment, release, rollback, approval, and operation status APIs.
- **Internal Dependencies:** Project Service, Pipeline Service, Build Service, Container Registry Service, Kubernetes Service, Monitoring Service, Audit Service, Scheduler Service.
- **Events Published:** Deployment requested, deployment approved, deployment started, deployment progressed, deployment completed, deployment failed, rollback started, rollback completed.
- **Events Consumed:** Build completed, image promoted, approval granted, cluster state changed, health gate result received, alert raised.
- **Storage Used:** PostgreSQL deployment/release records and operation metadata.
- **Cache Usage:** Active deployment cache, environment deployment summary cache, rollout status cache.
- **Security Considerations:** Production deployments require strict authorization, approval, audit, idempotency, and rollback controls.
- **Future Expansion:** Progressive delivery, canary analysis, deployment risk scoring, automated rollback, and multi-cluster orchestration.

## 10. Kubernetes Service

- **Purpose:** Manages Kubernetes cluster connections, runtime inventory, actions, and topology mapping.
- **Responsibilities:** Cluster registration, namespace/node/pod/container/service/ingress discovery, reconciliation, runtime action mediation, resource health, and topology edge publication.
- **Public Interfaces:** Cluster and runtime inventory APIs; controlled operational action APIs.
- **Internal Dependencies:** Integration Service, Deployment Service, Monitoring Service, Logging Service, Audit Service, secret manager.
- **Events Published:** Cluster connected, cluster disconnected, namespace observed, node observed, pod observed, container observed, service observed, ingress observed, runtime action completed.
- **Events Consumed:** Deployment started, cluster credential rotated, integration disconnected, topology refresh scheduled.
- **Storage Used:** PostgreSQL runtime inventory metadata; object storage for exported manifests where needed.
- **Cache Usage:** Hot runtime summary cache, cluster connection cache, topology node cache.
- **Security Considerations:** Mediate all cluster actions through audited service identities; never expose Kubernetes secrets; enforce environment and cluster permissions.
- **Future Expansion:** Multi-cluster policies, admission signals, runtime drift detection, workload recommendations, and operator integration.

## 11. Container Registry Service

- **Purpose:** Owns registry connections, image metadata, tags, digests, SBOM references, signing status, and scan summaries.
- **Responsibilities:** Registry integration metadata, image sync, artifact verification, image promotion, vulnerability summary ingestion, and image-to-build/deployment topology edges.
- **Public Interfaces:** Registry and image metadata APIs.
- **Internal Dependencies:** Integration Service, Build Service, Deployment Service, Audit Service, security scanners, secret manager.
- **Events Published:** Registry connected, registry disconnected, image observed, image promoted, image signed, image scan summarized.
- **Events Consumed:** Build image produced, registry webhook received, vulnerability scan completed, deployment requested.
- **Storage Used:** PostgreSQL registry/image metadata; object storage for SBOM and provenance documents.
- **Cache Usage:** Image digest lookup cache, registry connection cache, vulnerability summary cache.
- **Security Considerations:** Verify image digests, protect registry credentials, audit promotions and signature changes.
- **Future Expansion:** Policy-based promotion, attestations, registry mirroring, and advanced supply-chain graphing.

## 12. Monitoring Service

- **Purpose:** Owns observability configuration, metric metadata, health gates, alert correlation, and service health summaries.
- **Responsibilities:** Monitoring integration setup, metric query metadata, SLO references, deployment health evaluation, alert correlation, and topology health overlays.
- **Public Interfaces:** Monitoring configuration, health summary, metric query, and alert context APIs.
- **Internal Dependencies:** Integration Service, Deployment Service, Kubernetes Service, Alert/Notification flows, Audit Service.
- **Events Published:** Monitoring source connected, health gate passed, health gate failed, metric threshold breached, topology health changed.
- **Events Consumed:** Deployment started, deployment completed, cluster observed, alert rule changed, integration disconnected.
- **Storage Used:** PostgreSQL monitoring metadata and short-retention summaries; metrics backend for time series.
- **Cache Usage:** Health summary cache, SLO evaluation cache, metric query result cache with short TTL.
- **Security Considerations:** Metrics are tenant-scoped; avoid leaking labels containing secrets or cross-tenant topology.
- **Future Expansion:** Advanced SLOs, anomaly detection, release risk scoring, and AI-assisted diagnostics.

## 13. Logging Service

- **Purpose:** Owns log source metadata, log indexing envelopes, redaction policy, and log-to-topology correlation.
- **Responsibilities:** Log integration setup, log metadata indexing, trace/correlation mapping, deployment and pod log association, redaction state, and log export coordination.
- **Public Interfaces:** Log search, log metadata, log tailing, and log export APIs.
- **Internal Dependencies:** Integration Service, Kubernetes Service, Deployment Service, File Storage, Audit Service.
- **Events Published:** Log source connected, log envelope indexed, log export requested, sensitive log access recorded.
- **Events Consumed:** Pod observed, deployment started, trace observed, retention policy changed.
- **Storage Used:** PostgreSQL log metadata; external log storage or object storage for log bodies.
- **Cache Usage:** Recent log query cache, tail subscription state, redaction policy cache.
- **Security Considerations:** Redact secrets, audit sensitive log access, enforce tenant and project permissions.
- **Future Expansion:** Cross-signal correlation, AI log summarization, incident timelines, and retention tiering.

## 14. Notification Service

- **Purpose:** Delivers notifications to users, teams, and external channels.
- **Responsibilities:** Notification preferences, routing rules, templates, delivery attempts, provider dispatch, deduplication, and escalation coordination.
- **Public Interfaces:** Notification inbox, preferences, routing, and delivery status APIs.
- **Internal Dependencies:** User Service, Project Service, Integration Service, Audit Service, external providers.
- **Events Published:** Notification queued, notification delivered, notification failed, preference changed.
- **Events Consumed:** Alert raised, deployment failed, approval required, billing event occurred, AI recommendation ready, audit event matched notification policy.
- **Storage Used:** PostgreSQL notification records and delivery attempts.
- **Cache Usage:** Preference cache, routing cache, deduplication cache.
- **Security Considerations:** Do not leak sensitive payloads to channels; respect user and tenant notification policies.
- **Future Expansion:** On-call schedules, escalation policies, notification analytics, and chatops workflows.

## 15. AI Service

- **Purpose:** Provides governed AI conversations, recommendations, context retrieval, and tool orchestration.
- **Responsibilities:** AI conversation lifecycle, RAG/context retrieval, recommendation generation, tool policy checks, AI action proposals, evaluation metadata, and feedback capture.
- **Public Interfaces:** AI conversation, recommendation, context, and tool execution APIs.
- **Internal Dependencies:** Project Service, Repository Service, Deployment Service, Kubernetes Service, Monitoring Service, Logging Service, Audit Service, Integration Service, vector store/model providers.
- **Events Published:** AI conversation started, AI recommendation created, AI tool proposed, AI tool executed, AI feedback recorded.
- **Events Consumed:** Topology changed, deployment failed, alert raised, audit event recorded, documentation indexed, policy changed.
- **Storage Used:** PostgreSQL AI metadata; vector store for embeddings; object storage for evaluation artifacts where needed.
- **Cache Usage:** Context snippet cache, model response safety cache where approved, tool metadata cache.
- **Security Considerations:** Permission-filter all context, audit tool usage, never expose secrets, separate recommendation from execution, respect retention policy.
- **Future Expansion:** Agent workflows, incident copilots, policy-aware remediation, AI evaluations, and organization-specific model routing.

## 16. Audit Service

- **Purpose:** Owns immutable audit evidence for security, administrative, delivery, AI, billing, and operational actions.
- **Responsibilities:** Audit event intake, normalization, immutable storage, privileged audit querying, export workflows, retention, and compliance support.
- **Public Interfaces:** Audit query and export APIs for authorized users; internal audit ingestion contracts.
- **Internal Dependencies:** All domain services, object storage for exports, security policy components.
- **Events Published:** Audit record stored, audit export requested, audit export completed, audit retention action completed.
- **Events Consumed:** Domain audit events, authentication events, authorization denial events, privileged action events.
- **Storage Used:** Partitioned PostgreSQL audit log tables; object storage for archives and exports.
- **Cache Usage:** Minimal; short-lived export status or query acceleration only.
- **Security Considerations:** Audit logs are sensitive, immutable, access-controlled, and access to them is audited.
- **Future Expansion:** Compliance reports, anomaly detection, legal hold, and tenant-managed retention tiers.

## 17. Billing Service

- **Purpose:** Owns billing accounts, subscriptions, entitlements, usage rollups, invoices, and commercial lifecycle.
- **Responsibilities:** Subscription state, plan limits, entitlement checks, usage aggregation, billing provider sync, invoice metadata, and commercial audit events.
- **Public Interfaces:** Billing, subscription, entitlement, and usage APIs for authorized administrators.
- **Internal Dependencies:** Organization Service, Project Service, Audit Service, Integration Service, external billing provider.
- **Events Published:** Subscription created, subscription changed, entitlement changed, usage aggregated, invoice updated, payment status changed.
- **Events Consumed:** Organization created, usage event recorded, plan changed by provider, billing webhook received.
- **Storage Used:** PostgreSQL billing/subscription records and usage summaries.
- **Cache Usage:** Entitlement and quota cache with event-driven invalidation.
- **Security Considerations:** Financial data is sensitive; billing changes require privileged authorization and audit.
- **Future Expansion:** Usage-based billing, cost allocation, enterprise contracts, marketplace billing, and spend forecasting.

## 18. Integration Service

- **Purpose:** Owns external provider connections and integration metadata across Git, registry, cloud, identity, observability, notification, billing, and AI providers.
- **Responsibilities:** Provider connection lifecycle, credential references, scope metadata, health checks, provider account mapping, and integration policy.
- **Public Interfaces:** Integration connection, configuration, health, and disconnect APIs.
- **Internal Dependencies:** Authentication Service, Organization Service, Webhook Service, Audit Service, secret manager.
- **Events Published:** Integration connected, integration updated, integration unhealthy, integration disconnected, credential rotated.
- **Events Consumed:** Provider webhook received, organization suspended, credential rotation scheduled, provider health check completed.
- **Storage Used:** PostgreSQL integration metadata; secret manager for credentials.
- **Cache Usage:** Provider configuration cache, credential reference cache, health status cache.
- **Security Considerations:** Store only secret references; validate provider scopes; audit connection and scope changes.
- **Future Expansion:** Integration marketplace, provider-specific policy packs, customer-managed credentials, and regional provider routing.

## 19. Webhook Service

- **Purpose:** Receives, verifies, normalizes, and dispatches inbound provider webhooks.
- **Responsibilities:** Endpoint intake, signature verification, replay protection, payload normalization, deduplication, provider event persistence, and event bus publication.
- **Public Interfaces:** Provider webhook intake endpoints.
- **Internal Dependencies:** Integration Service, Repository Service, Container Registry Service, Billing Service, Event Bus, Audit Service.
- **Events Published:** Webhook received, webhook rejected, provider event normalized, provider event dispatched.
- **Events Consumed:** Integration connected, integration disconnected, webhook secret rotated.
- **Storage Used:** PostgreSQL webhook delivery metadata and deduplication records; object storage for large raw payloads when retention policy allows.
- **Cache Usage:** Signature key/reference cache, replay nonce cache, deduplication cache.
- **Security Considerations:** Verify signatures before trust, rate limit providers, prevent replay, avoid logging sensitive payloads.
- **Future Expansion:** Webhook replay UI, provider event debugging, schema evolution support, and customer-defined webhook routing.

## 20. Scheduler Service

- **Purpose:** Owns scheduled platform work, recurring jobs, delayed jobs, and time-based triggers.
- **Responsibilities:** Cron-like triggers, delayed operation scheduling, retention jobs, reconciliation schedules, pipeline schedules, credential rotation schedules, and billing aggregation schedules.
- **Public Interfaces:** Schedule management APIs where user-configurable; internal scheduling contracts.
- **Internal Dependencies:** Pipeline Service, Deployment Service, Kubernetes Service, Integration Service, Billing Service, Audit Service, Event Bus.
- **Events Published:** Schedule fired, scheduled job created, scheduled job updated, scheduled job failed.
- **Events Consumed:** Schedule configured, schedule disabled, organization suspended, retention policy changed.
- **Storage Used:** PostgreSQL schedule metadata and execution records.
- **Cache Usage:** Active schedule cache and distributed leadership/lock state in Redis where approved.
- **Security Considerations:** Scheduled jobs must run with recorded actor/service identity and tenant scope; production-impacting schedules require audit and policy checks.
- **Future Expansion:** Calendar-aware deployment windows, maintenance windows, tenant-local time policies, and workload-aware scheduling.
