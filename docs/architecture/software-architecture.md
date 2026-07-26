# SkyOps Software Architecture

## Purpose and architectural principles

SkyOps is an AI-powered DevOps operating system for enterprise platform engineering. The platform must unify repository operations, CI/CD, container image management, Kubernetes operations, infrastructure automation, observability, security, AI operations, and internal developer platform capabilities behind a cohesive API-first product.

The architecture is intentionally modular and domain-oriented so the product can scale from a startup codebase into a multi-tenant SaaS serving thousands of organizations and millions of deployments. Architectural decisions favor independently evolvable bounded contexts, clear ownership, explicit contracts, event-driven integration, and Kubernetes-native operations.

Core principles:

- **API first:** every capability is exposed through versioned contracts before UI or automation consumers depend on it.
- **Domain modularity:** business domains own their schemas, events, policies, and service contracts.
- **Event driven by default:** state changes emit durable events through NATS for workflows, audit, automation, and AI context building.
- **Cloud native:** services are containerized, deployed through Helm, and operated on Kubernetes.
- **Enterprise multi-tenancy:** organization, workspace, project, environment, and RBAC boundaries are first-class across every module.
- **AI ready:** AI services consume governed platform context through MCP tools, RAG indexes, event streams, and policy-controlled actions.
- **Long-term maintainability:** shared code is deliberately limited to stable primitives, generated clients, contracts, and platform libraries.

## 1. Recommended monorepo folder structure

```text
skyops/
  apps/
    web/
    admin/
    docs-site/
  services/
    identity/
    organizations/
    repositories/
    pipelines/
    artifacts/
    deployments/
    environments/
    kubernetes/
    infrastructure/
    observability/
    logging/
    security/
    notifications/
    audit/
    billing/
    ai-orchestrator/
    ai-rag/
    ai-tool-gateway/
    integrations/
    workflow-engine/
    event-router/
  workers/
    pipeline-runner/
    deployment-controller/
    drift-detector/
    metric-ingester/
    log-indexer/
    vulnerability-scanner/
    policy-evaluator/
    ai-context-builder/
  packages/
    contracts/
    config/
    authz/
    telemetry/
    events/
    errors/
    clients/
    go-kit/
    python-kit/
    ts-kit/
    testkit/
  infra/
    docker/
    kubernetes/
      base/
      overlays/
        local/
        dev/
        staging/
        production/
    helm/
      skyops/
      services/
    terraform/
      modules/
      environments/
        dev/
        staging/
        production/
    opentofu/
      modules/
      environments/
    policies/
    scripts/
  deploy/
    charts/
    manifests/
    gitops/
      clusters/
      tenants/
      apps/
  docs/
    architecture/
    adr/
    api/
    operations/
    security/
    ai/
    runbooks/
    product/
  tools/
    generators/
    linters/
    migrations/
    release/
  scripts/
  .github/
    workflows/
    ISSUE_TEMPLATE/
    PULL_REQUEST_TEMPLATE.md
```

Architectural decisions:

- `apps/` contains deployable user-facing applications only. This keeps product surfaces separate from backend domains.
- `services/` contains independently deployable backend bounded contexts. Go services should be the default for platform control-plane workloads; Python services should be used only where AI, ML, embedding, retrieval, or model integration ergonomics justify it.
- `workers/` contains asynchronous processors and controllers that react to events, scheduled jobs, or queues. Workers are isolated from synchronous APIs to protect API latency and availability.
- `packages/` contains shared libraries and generated artifacts. It must not become a place for hidden business logic.
- `infra/` contains infrastructure source definitions, reusable Terraform/OpenTofu modules, Helm charts, policies, and cluster configuration.
- `deploy/` contains deployment orchestration assets, GitOps application definitions, and environment-specific release bindings.
- `docs/` is treated as a product-critical artifact. Architecture, API contracts, operations, security, AI behavior, and runbooks are versioned with code.
- `tools/` contains repository automation that supports generation, linting, migrations, and releases without coupling to runtime code.

## 2. Module boundaries

SkyOps should be organized around durable platform domains rather than technical layers.

### Identity and access

Owns authentication integration, service identities, users, groups, roles, permissions, sessions, API tokens, SSO, SCIM, and organization-level access controls.

Boundaries:

- Owns authorization policy primitives and identity lifecycle events.
- Does not own product-specific permission decisions embedded in individual domains.
- Publishes identity events consumed by audit, billing, notification, and AI-context services.

### Organizations and tenancy

Owns organizations, workspaces, projects, tenant lifecycle, tenant limits, region placement, and enterprise account settings.

Boundaries:

- Every other domain references organization, workspace, and project identifiers but does not mutate tenant ownership state.
- Provides tenant metadata required for routing, quotas, encryption configuration, and regional isolation.

### Source repositories

Owns Git provider connections, repository metadata, branch protections, webhook intake, commit metadata, pull request metadata, and repository-to-project mapping.

Boundaries:

- Does not execute CI/CD workloads.
- Emits repository and source events for pipeline triggers, AI summarization, security scanning, and audit.

### CI/CD pipelines

Owns pipeline definitions, pipeline runs, stages, jobs, execution graph state, approvals, retries, and run history.

Boundaries:

- Delegates execution to `pipeline-runner` workers.
- Consumes repository events and artifact events.
- Emits lifecycle events for deployments, notifications, audit, and AI operations.

### Artifacts and images

Owns Docker image metadata, build artifacts, SBOM references, provenance, signing status, promotion state, and registry integrations.

Boundaries:

- Does not own deployment orchestration.
- Provides artifact verification data to deployments and security modules.

### Deployments and releases

Owns release records, deployment plans, rollouts, approvals, promotion between environments, rollback history, and deployment health summaries.

Boundaries:

- Delegates Kubernetes actions to the Kubernetes domain.
- Uses observability and logging domains for health gates.
- Emits deployment events for audit, notifications, AI analysis, and billing usage.

### Environments

Owns logical environments such as development, staging, production, ephemeral preview environments, cluster bindings, protection rules, and environment-specific variables.

Boundaries:

- Does not own raw secret values.
- Supplies environment context to pipelines, deployments, infrastructure, and policy evaluation.

### Kubernetes operations

Owns cluster connections, namespace mapping, workload inventory, reconciliation status, resource actions, policy admission signals, and Kubernetes API abstractions.

Boundaries:

- Kubernetes operations are mediated through controllers and audited service identities.
- Other domains request desired operations; this module validates and applies them.

### Infrastructure automation

Owns infrastructure stacks, Terraform/OpenTofu runs, drift detection, state backend metadata, plan approvals, and cloud account integration metadata.

Boundaries:

- Does not store cloud provider secrets directly; it references secret providers and short-lived credentials.
- Emits plan, apply, destroy, and drift events.

### Observability and logging

Owns metrics metadata, service health, alert definitions, SLOs, log stream metadata, trace indexes, dashboards, and incident signals.

Boundaries:

- Prometheus, Grafana, Loki, and OpenTelemetry remain infrastructure components; SkyOps stores product-level metadata and access mappings.
- Provides signals to deployment gates, AI operations, security, and incident workflows.

### Security and compliance

Owns vulnerability findings, policy definitions, compliance controls, evidence records, security posture, secret scanning results, and exception workflows.

Boundaries:

- Provides policy decisions to pipelines, deployments, repositories, and infrastructure.
- Emits security findings for audit, notification, and AI-assisted remediation.

### AI operations

Owns AI assistants, tool policies, MCP server registry, prompt templates, RAG collections, embedding jobs, model routing, reasoning traces, and AI action approvals.

Boundaries:

- AI services may recommend or request actions but must not bypass domain APIs, RBAC, audit, or approval gates.
- Context is retrieved through governed APIs, event-derived indexes, and MCP tools rather than direct database access to every service.

### Workflow and automation

Owns cross-domain workflows, durable orchestration, scheduled jobs, automations, approval flows, and compensating actions.

Boundaries:

- Coordinates domains through APIs and events.
- Does not own domain records that belong to another bounded context.

## 3. Microservices

Recommended service portfolio:

| Service | Runtime | Primary responsibility | Data ownership |
| --- | --- | --- | --- |
| `identity-service` | Go | AuthN/AuthZ, users, groups, service accounts, SSO, SCIM | Identity database schema |
| `organization-service` | Go | Tenants, workspaces, projects, quotas, account settings | Organization schema |
| `repository-service` | Go | Git provider integrations, repository metadata, webhooks | Repository schema |
| `pipeline-service` | Go | Pipeline definitions, run state, approvals | Pipeline schema |
| `artifact-service` | Go | Images, artifacts, SBOM, provenance, signing metadata | Artifact schema |
| `deployment-service` | Go | Releases, rollouts, promotions, rollback state | Deployment schema |
| `environment-service` | Go | Environments, variables metadata, protection rules | Environment schema |
| `kubernetes-service` | Go | Cluster inventory, namespace/workload views, API mediation | Kubernetes metadata schema |
| `infrastructure-service` | Go | Terraform/OpenTofu stacks, runs, plans, drift state | Infrastructure schema |
| `observability-service` | Go | Metrics metadata, SLOs, alert rules, health summaries | Observability schema |
| `logging-service` | Go | Log source metadata, saved queries, retention policies | Logging schema |
| `security-service` | Go | Findings, policies, compliance evidence, exceptions | Security schema |
| `notification-service` | Go | Email, Slack, webhook, incident notification routing | Notification schema |
| `audit-service` | Go | Immutable audit event records and query APIs | Audit schema or append-only store |
| `billing-service` | Go | Plans, subscriptions, metering, usage aggregation | Billing schema |
| `integration-service` | Go | Third-party provider connectors and OAuth installations | Integration schema |
| `workflow-engine` | Go | Durable cross-domain workflows and automations | Workflow schema |
| `event-router` | Go | Event normalization, subscriptions, replay, fanout | Event metadata schema |
| `ai-orchestrator` | Python | AI session orchestration, model routing, guardrails | AI operations schema |
| `ai-rag-service` | Python | Embeddings, retrieval, collection lifecycle | RAG metadata schema plus vector backend |
| `ai-tool-gateway` | Python or Go | MCP tool registry, tool invocation policy, action mediation | Tool invocation schema |

Architectural decisions:

- Services map to bounded contexts so teams can own them independently.
- Each service owns its persistence schema. Cross-service joins are avoided in favor of APIs, projections, and events.
- Go is the default for control-plane services because it is efficient, reliable, and common in Kubernetes ecosystems.
- Python is reserved for AI-specific services where model libraries, embedding tooling, and RAG ecosystem support are stronger.
- Workers are separately deployable so long-running and bursty operations do not threaten user-facing APIs.

## 4. Shared packages

Shared packages should accelerate consistency without creating a distributed monolith.

Recommended packages:

- `packages/contracts`: OpenAPI, AsyncAPI, JSON Schema, Protobuf, and event contract definitions.
- `packages/config`: typed configuration loading conventions, environment variable names, and validation helpers.
- `packages/authz`: authorization primitives, policy request/response structures, and shared permission constants.
- `packages/telemetry`: OpenTelemetry conventions, span attributes, metrics labels, and logging fields.
- `packages/events`: event envelope definitions, subject naming, schema registry helpers, and idempotency metadata.
- `packages/errors`: common error taxonomy, problem details structures, retry classifications, and user-safe error codes.
- `packages/clients`: generated service clients from API contracts.
- `packages/go-kit`: Go middleware for logging, tracing, metrics, NATS, PostgreSQL, Redis, health checks, and graceful shutdown.
- `packages/python-kit`: Python utilities for AI services, tracing, config, model routing adapters, and MCP invocation policies.
- `packages/ts-kit`: TypeScript API clients, shared validation schemas, telemetry helpers, and frontend-safe constants.
- `packages/testkit`: contract testing utilities, fake providers, event fixtures, and integration test harnesses.

Architectural decisions:

- Shared packages may contain stable cross-cutting primitives but not domain workflows.
- Generated clients prevent contract drift and reduce hand-written integration code.
- Telemetry, error, event, and config packages enforce consistent production behavior across services.

## 5. Infrastructure folders

Infrastructure should be organized for local development, SaaS environments, customer-managed deployments, and future regional expansion.

Recommended layout:

- `infra/docker`: local Dockerfiles, Compose profiles, and base images used only for development and CI.
- `infra/kubernetes/base`: common Kubernetes manifests that apply to all environments.
- `infra/kubernetes/overlays`: Kustomize-style environment overlays for local, dev, staging, and production.
- `infra/helm/skyops`: umbrella chart for deploying the platform.
- `infra/helm/services`: service-specific charts or chart templates.
- `infra/terraform/modules`: reusable cloud infrastructure modules.
- `infra/terraform/environments`: environment instantiations, remote state, and variable bindings.
- `infra/opentofu`: parallel structure for OpenTofu compatibility where required by customer or licensing needs.
- `infra/policies`: OPA, Kyverno, admission, security, and compliance policies.
- `deploy/gitops`: Argo CD or Flux application definitions for clusters, tenants, and applications.

Architectural decisions:

- Helm provides product packaging; Terraform/OpenTofu provisions cloud resources; GitOps reconciles desired state.
- Environment overlays prevent production-specific configuration from leaking into local development.
- Policies are versioned with infrastructure to make compliance review auditable.

## 6. Documentation folders

Recommended documentation structure:

- `docs/architecture`: system architecture, domain maps, diagrams, data ownership, integration patterns.
- `docs/adr`: Architecture Decision Records using monotonically increasing filenames such as `0001-use-nats-for-events.md`.
- `docs/api`: OpenAPI, AsyncAPI, auth flows, idempotency rules, pagination standards, error standards.
- `docs/operations`: deployment, scaling, backup, restore, upgrade, incident response, capacity planning.
- `docs/security`: threat models, access control model, data classification, encryption, compliance controls.
- `docs/ai`: AI governance, model routing, RAG design, MCP tool policy, prompt lifecycle, evaluation strategy.
- `docs/runbooks`: executable operational procedures for common incidents and maintenance tasks.
- `docs/product`: product capability maps, user journeys, terminology, and enterprise readiness requirements.

Architectural decisions:

- Architecture decisions are recorded as ADRs so future teams understand why trade-offs were made.
- API and event documentation is maintained beside contracts so consumer expectations remain clear.
- Security, AI, and operations documentation are first-class because they are enterprise purchasing requirements.

## 7. Naming conventions

Repository naming:

- Root repository: `skyops`.
- Services: `<domain>-service`, for example `pipeline-service`.
- Workers: `<domain>-<purpose>-worker` or `<purpose>-controller`, for example `pipeline-runner` and `deployment-controller`.
- Packages: `@skyops/<name>` for TypeScript packages, `github.com/skyops/skyops/packages/<name>` for Go packages, and `skyops_<name>` for Python packages.

API naming:

- REST resources use plural nouns: `/v1/organizations/{organizationId}/projects`.
- IDs are explicit and scoped: `organizationId`, `workspaceId`, `projectId`, `environmentId`.
- Mutating operations that are not simple CRUD use command-style subresources: `/v1/deployments/{deploymentId}:rollback`.

Event naming:

- NATS subjects use the pattern `skyops.<environment>.<tenant-region>.<domain>.<entity>.<event-version>.<event-name>`.
- Example: `skyops.production.us-east-1.deployments.release.v1.completed`.
- Event names use past-tense facts: `created`, `updated`, `approved`, `failed`, `completed`, `revoked`.

Database naming:

- Tables use plural snake_case names.
- Columns use snake_case.
- Primary keys use `<entity>_id`.
- Tenant-scoped tables include `organization_id` and usually `project_id`.

Kubernetes naming:

- Namespaces use `skyops-<environment>` for platform namespaces and `skyops-tenant-<tenant-slug>` only when tenant isolation requires dedicated namespaces.
- Kubernetes resources use labels: `app.kubernetes.io/name`, `app.kubernetes.io/component`, `app.kubernetes.io/part-of`, `skyops.io/domain`, and `skyops.io/environment`.

## 8. Coding standards

General standards:

- All services must expose health, readiness, metrics, tracing, structured logs, and version endpoints.
- Every request must carry correlation IDs, tenant context, actor context, and authorization context.
- All external calls must use timeouts, retries with backoff where safe, circuit breakers for unstable dependencies, and idempotency keys for mutating operations.
- All service-to-service APIs must be contract-tested.
- Direct database access across service boundaries is forbidden.
- Public and internal APIs must return stable error codes and user-safe messages.
- Migrations must be backward compatible across rolling deployments.
- Secrets must never be logged, embedded in code, stored in Git, or exposed through AI context.

Go standards:

- Prefer small packages with explicit interfaces at boundaries.
- Use context propagation for cancellation, deadlines, tracing, and request-scoped metadata.
- Keep domain logic separate from transport handlers and persistence adapters.
- Use generated clients for cross-service calls.

Python AI standards:

- Keep model adapters, retrieval, guardrails, and tool execution separated.
- Log AI decisions with redaction, trace IDs, model identifiers, prompt versions, and policy decisions.
- Require approval gates for high-impact actions such as deployment rollback, infrastructure apply, access changes, and secret rotation.

TypeScript standards:

- Use generated API clients and shared validation schemas.
- Keep application state, API access, and presentation components separated.
- Avoid duplicating authorization rules in the frontend; the backend remains the source of truth.

## 9. Branch strategy

Recommended strategy: trunk-based development with protected release branches.

- `main`: always releasable and protected.
- `feature/<short-description>`: short-lived feature branches merged through pull requests.
- `fix/<short-description>`: short-lived defect branches.
- `chore/<short-description>`: maintenance and repository hygiene.
- `release/<major>.<minor>`: stabilization branch for a release train when needed.
- `hotfix/<version>`: emergency production fixes branched from the affected release tag or branch.

Architectural decisions:

- Trunk-based development minimizes long-lived divergence and supports continuous delivery.
- Release branches exist only when enterprise support or staged rollout requirements require stabilization windows.
- All merges to `main` require CI, tests, code review, security scanning, and contract compatibility checks.

## 10. Versioning strategy

SkyOps should use SemVer for product releases and explicit versioning for APIs, events, charts, and infrastructure modules.

- Product versions use `MAJOR.MINOR.PATCH`.
- API versions are path-based for externally consumed APIs, such as `/v1`.
- Event schemas are versioned in the subject and schema metadata, such as `release.v1.completed`.
- Helm charts use independent chart versions and app versions.
- Terraform/OpenTofu modules use immutable version tags.
- Service container images are tagged with Git SHA, SemVer release tag when applicable, and build metadata.
- Database migrations are monotonic and immutable after merge.

Architectural decisions:

- Independent service deployment requires backward-compatible contracts.
- Event versioning prevents asynchronous consumers from breaking during phased rollouts.
- Immutable infrastructure module versions support reproducibility and enterprise audits.

## 11. Environment strategy

Recommended environments:

- `local`: developer machines using Docker Compose, kind, or k3d.
- `dev`: shared integration environment with non-production data.
- `staging`: production-like validation environment with production topology and synthetic data.
- `production`: customer-facing SaaS environment.
- `preview`: ephemeral environments per pull request or product branch.
- `sandbox`: optional isolated environment for sales engineering, demos, and customer trials.

Environment rules:

- Production must be isolated by cloud account, cluster, database, secrets, and network boundaries.
- Staging should mirror production topology closely enough to validate scaling, migrations, observability, and rollback.
- Preview environments should be cost-controlled, time-limited, and seeded with synthetic data.
- Local development should run a minimal platform slice without requiring cloud credentials.
- AI providers, model configuration, and tool permissions must vary by environment.

Architectural decisions:

- Environment parity reduces production-only failures.
- Strict production isolation supports enterprise compliance and incident containment.
- Preview environments improve product quality while limiting risk to shared environments.

## 12. Configuration strategy

Configuration should follow a layered model:

1. Code defaults for safe local behavior.
2. Environment-level configuration from ConfigMaps, Helm values, or deployment manifests.
3. Secret values from a secrets manager or Kubernetes secrets sealed through an approved mechanism.
4. Tenant-level settings stored through domain services.
5. Runtime feature flags controlled by an audited feature flag system.

Standards:

- Configuration keys must be documented in `packages/config`.
- Services validate configuration at startup and fail fast on invalid required settings.
- Secrets are referenced, not copied, across domains.
- Feature flags must include owner, expiry date, environment scope, and rollback instructions.
- AI configuration must include provider, model, context limits, tool permissions, data-retention policy, and safety policy.

Architectural decisions:

- Layered configuration separates deploy-time, secret, tenant, and runtime concerns.
- Strong startup validation prevents latent production misconfiguration.
- Feature flag governance prevents stale flags from becoming permanent hidden architecture.

## 13. Dependency management strategy

Dependency standards:

- Use workspace-aware dependency management for JavaScript and TypeScript packages.
- Use Go modules per service or per bounded service group, with generated clients pinned by version.
- Use Python lockfiles for AI services and workers.
- Centralize base container images and scan them continuously.
- Pin infrastructure providers and modules to explicit versions.
- Require automated dependency update pull requests with CI, tests, SBOM generation, and vulnerability scanning.
- Maintain an allowlist for critical runtime libraries and platform dependencies.
- Generate SBOMs for each service image and release artifact.

Architectural decisions:

- Explicit pinning improves reproducibility.
- Automated updates reduce security lag without relying on manual dependency audits.
- SBOMs and image scanning are required for enterprise procurement, incident response, and compliance.

## 14. High-level architecture diagram

```text
+--------------------------------------------------------------------------------+
|                                  Users & Teams                                  |
|  Developers | Platform Engineers | SREs | Security | Executives | AI Operators  |
+----------------------------------------+---------------------------------------+
                                         |
                                         v
+--------------------------------------------------------------------------------+
|                              Applications Layer                                |
|        apps/web          apps/admin          docs-site          CLI/Future SDKs |
+----------------------------------------+---------------------------------------+
                                         |
                                         v
+--------------------------------------------------------------------------------+
|                                API Gateway                                     |
|     AuthN/AuthZ | Rate Limits | Tenant Routing | Request Logging | API Versions |
+----------------------------------------+---------------------------------------+
                                         |
                                         v
+--------------------------------------------------------------------------------+
|                              Domain Services                                   |
| Identity | Orgs | Repos | Pipelines | Artifacts | Deployments | Environments   |
| Kubernetes | Infrastructure | Observability | Logging | Security | Billing      |
| Notifications | Audit | Integrations | Workflow Engine | Event Router        |
+----------------------------------------+---------------------------------------+
            |                            |                              |
            | synchronous APIs           | domain events                 | workflows
            v                            v                              v
+-----------------------------+   +------------------------------+   +-------------+
| PostgreSQL                  |   | NATS                         |   | Workers     |
| Service-owned schemas       |   | JetStream durable streams    |   | Controllers |
| Tenant-aware records        |   | AsyncAPI event contracts     |   | Scanners    |
+-----------------------------+   +------------------------------+   +-------------+
            |                            |                              |
            v                            v                              v
+--------------------------------------------------------------------------------+
|                            Platform Capabilities                               |
| Redis | Prometheus | Grafana | Loki | OpenTelemetry | Secrets | Policy Engine  |
+----------------------------------------+---------------------------------------+
                                         |
                                         v
+--------------------------------------------------------------------------------+
|                                  AI Layer                                      |
| AI Orchestrator | RAG Service | MCP Tool Gateway | Model Providers | Ollama      |
| Context Builder | Guardrails | Human Approval | Audit-Aware Actions              |
+----------------------------------------+---------------------------------------+
                                         |
                                         v
+--------------------------------------------------------------------------------+
|                           External Systems                                     |
| Git Providers | Container Registries | Kubernetes Clusters | Clouds | Slack      |
| Terraform/OpenTofu Backends | OpenAI-compatible APIs | Customer Systems             |
+--------------------------------------------------------------------------------+
```

Architectural decisions:

- The API gateway centralizes cross-cutting edge concerns but does not contain domain logic.
- Domain services communicate synchronously for direct user requests and asynchronously through NATS for lifecycle events.
- Workers handle execution-heavy tasks such as pipeline jobs, deployments, scans, drift checks, indexing, and notifications.
- AI services sit behind policy and audit boundaries and invoke domain capabilities through approved tools and APIs.
- Observability is part of the platform foundation and feeds both human operations and AI-assisted operations.

## 15. Development workflow

Recommended workflow:

1. Create or update an ADR for significant architectural decisions.
2. Define or update API and event contracts before implementation.
3. Generate clients and schemas from contracts.
4. Implement domain logic inside the owning service or worker.
5. Add unit tests, integration tests, contract tests, migration tests, and security checks.
6. Run local development with Docker Compose or a local Kubernetes cluster.
7. Open a pull request with architecture, contract, implementation, migration, and operational notes.
8. CI validates formatting, linting, tests, dependency checks, vulnerability scans, generated-code freshness, and contract compatibility.
9. Merge to `main` after review and green CI.
10. Deploy automatically to `dev`.
11. Promote to `staging` after integration checks.
12. Promote to `production` through progressive delivery, health gates, and rollback readiness.
13. Post-deployment events update audit logs, release records, metrics, and AI context indexes.

Architectural decisions:

- Contract-first development prevents frontend, backend, worker, and AI consumers from diverging.
- Progressive promotion reduces production risk.
- Every deployment leaves operational evidence through audit events, telemetry, release metadata, and runbooks.
- AI context is built from governed platform events and APIs so recommendations stay current without violating tenant or security boundaries.
