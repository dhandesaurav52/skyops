# SkyOps Production Monorepo Design

## Purpose

This document defines the complete production-ready monorepo structure for SkyOps. The goal is to provide a repository layout that can support a decade of growth across frontend applications, Go backend services, Python AI services, shared libraries, SDKs, CLI tooling, infrastructure-as-code, Kubernetes operations, CI/CD, testing, documentation, examples, and enterprise delivery workflows.

This document intentionally defines repository structure only. It does not define implementation code, UI, APIs, database schemas, or business logic.

## Design principles

- **Domain-oriented organization:** product capabilities live near their owning domain instead of being scattered by technical concern.
- **Clear deployable boundaries:** every deployable application, service, worker, extension, or tool has an obvious home.
- **Contract-first growth:** shared contracts and generated SDKs have first-class locations so teams can scale without copy-paste integration code.
- **Infrastructure as product:** Docker, Kubernetes, Helm, Terraform, OpenTofu, policy, and GitOps assets are versioned and reviewed like application architecture.
- **AI-native but governed:** prompt libraries, RAG assets, MCP definitions, evaluations, and AI policies are centralized enough to govern but modular enough to evolve.
- **Testing at multiple layers:** unit, contract, integration, end-to-end, load, security, migration, and chaos tests all have explicit homes.
- **Tooling consistency:** scripts, generators, linters, release automation, and developer tooling are discoverable and reusable.
- **Long-term maintainability:** shared packages stay small, stable, generated where possible, and free of hidden domain ownership.

## 1. Complete root folder structure

```text
skyops/
  apps/
    web/
    admin/
    console/
    docs-site/
    marketing-site/
    vscode-extension/
  services/
    go/
      identity-service/
      organization-service/
      repository-service/
      pipeline-service/
      artifact-service/
      deployment-service/
      environment-service/
      kubernetes-service/
      infrastructure-service/
      observability-service/
      logging-service/
      security-service/
      notification-service/
      audit-service/
      billing-service/
      integration-service/
      workflow-engine/
      event-router/
    python/
      ai-orchestrator/
      ai-rag-service/
      ai-tool-gateway/
      ai-evaluation-service/
  workers/
    pipeline-runner/
    deployment-controller/
    drift-detector/
    metric-ingester/
    log-indexer/
    vulnerability-scanner/
    policy-evaluator/
    ai-context-builder/
    notification-dispatcher/
  packages/
    contracts/
    events/
    config/
    authz/
    telemetry/
    errors/
    clients/
    schemas/
    feature-flags/
    go-kit/
    python-kit/
    ts-kit/
    testkit/
  sdks/
    typescript/
    go/
    python/
  cli/
    skyops/
  ai/
    prompts/
      system/
      agents/
      workflows/
      evaluations/
      guardrails/
    rag/
      collections/
      ingestion/
      policies/
    mcp/
      servers/
      tools/
      policies/
    evals/
  infrastructure/
    docker/
    kubernetes/
    helm/
    terraform/
    opentofu/
    policies/
    secrets/
    observability/
    networking/
  deployments/
    gitops/
      clusters/
      tenants/
      apps/
    environments/
      local/
      dev/
      staging/
      production/
      preview/
      sandbox/
    release/
  docker/
    base-images/
    compose/
    devcontainers/
    service-images/
  kubernetes/
    base/
    overlays/
      local/
      dev/
      staging/
      production/
    operators/
    crds/
    admission/
  terraform/
    modules/
    environments/
      dev/
      staging/
      production/
    providers/
    backends/
  opentofu/
    modules/
    environments/
      dev/
      staging/
      production/
    providers/
    backends/
  configs/
    eslint/
    typescript/
    prettier/
    tailwind/
    go/
    python/
    markdown/
    commitlint/
    renovate/
    security/
    telemetry/
  scripts/
    bootstrap/
    dev/
    ci/
    release/
    migrations/
    security/
    maintenance/
  tools/
    generators/
    linters/
    codegen/
    contract-testing/
    load-testing/
    migration-tools/
    release-tools/
    local-dev/
  tests/
    unit/
    integration/
    contract/
    e2e/
    performance/
    security/
    chaos/
    migration/
    fixtures/
  docs/
    architecture/
    adr/
    api/
    operations/
    security/
    ai/
    runbooks/
    platform/
    product/
    contributing/
  examples/
    pipelines/
    deployments/
    infrastructure/
    kubernetes/
    ai-workflows/
    integrations/
    sdk-usage/
  .github/
    workflows/
    actions/
    ISSUE_TEMPLATE/
    PULL_REQUEST_TEMPLATE.md
    CODEOWNERS
  .devcontainer/
  .editorconfig
  .gitignore
  README.md
```

The root is intentionally split between product deliverables, shared building blocks, platform operations, repository tooling, tests, examples, and governance. This keeps the project navigable as the number of teams and modules grows.

## 2. `apps/`

`apps/` contains user-facing applications and extensions. These are deployable or distributable products, not shared libraries.

Recommended folders:

- `apps/web`: primary customer-facing Next.js application for platform engineers, developers, SREs, and security teams.
- `apps/admin`: internal or enterprise administration application for support, tenant management, billing operations, and compliance support workflows.
- `apps/console`: optional lightweight operational console for self-hosted, air-gapped, or cluster-local administration.
- `apps/docs-site`: documentation website generated from `docs/` content.
- `apps/marketing-site`: public marketing site if the company chooses to keep it in the same monorepo.
- `apps/vscode-extension`: VS Code extension for repository, pipeline, deployment, and AI-assisted developer workflows.

Why it exists:

- Separates product surfaces from backend services and libraries.
- Allows each app to have independent routing, build, release, telemetry, and environment configuration.
- Prevents frontend-specific dependencies from leaking into services or shared backend packages.

Placement rules:

- New web UI surfaces live under an existing app unless they are independently deployed or distributed.
- Browser-only utilities live in `packages/ts-kit`, not inside a single app, when they are shared by more than one app.
- Extension-specific assets live in `apps/vscode-extension` unless they are generated SDKs or shared schemas.

## 3. `services/`

`services/` contains synchronous backend and AI service boundaries. Services own product capabilities and persistence boundaries.

Recommended folders:

- `services/go`: Go control-plane services for core SaaS operations.
- `services/python`: Python services where AI, ML, embeddings, model adapters, or RAG ecosystem support justify Python.

Go service folders:

- `identity-service`: authentication, users, groups, service accounts, SSO, SCIM, sessions, and API tokens.
- `organization-service`: organizations, workspaces, projects, quotas, and tenant settings.
- `repository-service`: Git provider installation metadata, repository mapping, webhook intake, and source events.
- `pipeline-service`: pipeline definitions, run orchestration state, approvals, retries, and history.
- `artifact-service`: build artifacts, container image metadata, SBOM references, provenance, and signing status.
- `deployment-service`: releases, deployment plans, promotions, rollbacks, and release health summaries.
- `environment-service`: logical environments, protection rules, cluster bindings, and variable metadata.
- `kubernetes-service`: cluster inventory, workload status, namespace mapping, and mediated Kubernetes operations.
- `infrastructure-service`: Terraform/OpenTofu stack metadata, run state, drift status, plans, and approvals.
- `observability-service`: metrics metadata, SLOs, alert definitions, dashboard metadata, and health summaries.
- `logging-service`: log source metadata, retention settings, saved queries, and log access mapping.
- `security-service`: vulnerability findings, policy records, compliance evidence, exceptions, and posture summaries.
- `notification-service`: email, Slack, webhook, incident routing, templates, and notification preferences.
- `audit-service`: immutable audit ingestion, retention, export, and audit query APIs.
- `billing-service`: subscription plans, usage metering, invoice metadata, and entitlements.
- `integration-service`: third-party provider installations, OAuth connections, and external system mappings.
- `workflow-engine`: durable cross-domain workflows, approvals, schedules, and compensating actions.
- `event-router`: event normalization, subscriptions, replay metadata, and fanout rules.

Python service folders:

- `ai-orchestrator`: AI sessions, model routing, planner behavior, guardrails, and AI action mediation.
- `ai-rag-service`: embeddings, retrieval indexes, collection lifecycle, document chunking policy, and context search.
- `ai-tool-gateway`: MCP tool registry, tool invocation policy, audit-aware tool execution, and action approval checks.
- `ai-evaluation-service`: AI prompt evaluation, regression testing, offline scoring, and safety evaluations.

Why it exists:

- Keeps independently deployable service ownership explicit.
- Makes it clear which language runtime is appropriate for each domain.
- Supports service-level scaling, ownership, CI, deployment, and incident response.

Placement rules:

- A new domain service lives in `services/go` by default.
- A new AI/model-centric service lives in `services/python` only when Python materially improves the domain.
- Domain-specific business rules live with the owning service, never in `packages/`.

## 4. `packages/`

`packages/` contains shared building blocks that are stable, intentionally cross-cutting, or generated from contracts.

Recommended folders:

- `packages/contracts`: OpenAPI, AsyncAPI, JSON Schema, Protobuf, and service contract source files.
- `packages/events`: event envelope definitions, NATS subject conventions, schema metadata, and idempotency primitives.
- `packages/config`: typed configuration conventions, environment variable names, validation patterns, and defaults catalog.
- `packages/authz`: permission constants, policy request/response structures, and authorization helper primitives.
- `packages/telemetry`: OpenTelemetry conventions, metrics labels, log fields, trace attributes, and correlation standards.
- `packages/errors`: common error taxonomy, retry classification, user-safe error codes, and problem-details conventions.
- `packages/clients`: generated internal clients for service-to-service communication.
- `packages/schemas`: shared validation schemas that are not owned by a single API contract.
- `packages/feature-flags`: feature flag metadata, naming conventions, lifecycle policy, and generated flag catalogs.
- `packages/go-kit`: Go platform utilities for transport, telemetry, NATS, PostgreSQL, Redis, health checks, and graceful shutdown.
- `packages/python-kit`: Python platform utilities for AI services, model adapters, tracing, config, and MCP policy handling.
- `packages/ts-kit`: TypeScript utilities, frontend-safe constants, generated schemas, and shared app helpers.
- `packages/testkit`: fixtures, fake providers, test containers, contract test harnesses, and golden event samples.

Why it exists:

- Reduces duplication while preserving service autonomy.
- Centralizes generated artifacts and cross-cutting platform primitives.
- Provides consistency for telemetry, errors, configuration, and events.

Placement rules:

- Shared packages must not own product workflows.
- A package should exist only when at least two consumers need the same stable primitive.
- Generated code belongs under `packages/clients` or an SDK folder, not inside individual services.

## 5. `infrastructure/`

`infrastructure/` contains platform infrastructure architecture and reusable operational assets.

Recommended folders:

- `infrastructure/docker`: canonical Docker architecture, base image policy, image hardening guidance, and shared image metadata.
- `infrastructure/kubernetes`: cluster architecture, namespace model, admission policy, service mesh decisions, and controller deployment patterns.
- `infrastructure/helm`: Helm chart architecture and shared chart conventions.
- `infrastructure/terraform`: Terraform module design, environment composition, state strategy, and provider conventions.
- `infrastructure/opentofu`: OpenTofu equivalents for customers or environments that require OpenTofu.
- `infrastructure/policies`: OPA, Kyverno, admission, compliance, and deployment guardrail policies.
- `infrastructure/secrets`: secret management strategy, external secret references, rotation policy, and sealed secret conventions.
- `infrastructure/observability`: Prometheus, Grafana, Loki, OpenTelemetry, alerting, dashboards, and SLO infrastructure.
- `infrastructure/networking`: ingress, service mesh, DNS, private networking, egress, and zero-trust network design.

Why it exists:

- Treats infrastructure as a product capability rather than an afterthought.
- Creates a stable home for platform-wide operational decisions.
- Allows implementation-specific folders such as `terraform/` and `kubernetes/` to stay focused on deployable assets.

## 6. `deployments/`

`deployments/` contains environment-specific release bindings and GitOps desired state.

Recommended folders:

- `deployments/gitops/clusters`: cluster-level GitOps definitions.
- `deployments/gitops/tenants`: tenant-specific GitOps definitions when dedicated tenant infrastructure exists.
- `deployments/gitops/apps`: application and service deployment bindings.
- `deployments/environments/local`: local stack composition.
- `deployments/environments/dev`: shared development deployment inputs.
- `deployments/environments/staging`: production-like deployment inputs.
- `deployments/environments/production`: production deployment inputs with strict controls.
- `deployments/environments/preview`: ephemeral pull-request deployment inputs.
- `deployments/environments/sandbox`: sales engineering, demo, and trial environment inputs.
- `deployments/release`: release manifests, promotion metadata, rollout plans, and rollback records.

Why it exists:

- Separates deploy-time environment binding from reusable infrastructure modules and charts.
- Makes GitOps ownership explicit.
- Supports progressive delivery, preview environments, and production promotion workflows.

## 7. `docker/`

`docker/` contains Docker assets used for local development, CI, and image standardization.

Recommended folders:

- `docker/base-images`: hardened base image definitions and metadata.
- `docker/compose`: Docker Compose profiles for local platform slices.
- `docker/devcontainers`: developer container definitions.
- `docker/service-images`: shared service image templates and build conventions.

Why it exists:

- Keeps Docker-specific artifacts discoverable outside application code.
- Standardizes local development and CI image behavior.
- Provides a controlled path for image hardening and vulnerability management.

## 8. `kubernetes/`

`kubernetes/` contains raw Kubernetes deployment assets that are not Terraform modules or Helm chart templates.

Recommended folders:

- `kubernetes/base`: common manifests for platform services.
- `kubernetes/overlays/local`: local cluster overlays.
- `kubernetes/overlays/dev`: development cluster overlays.
- `kubernetes/overlays/staging`: staging cluster overlays.
- `kubernetes/overlays/production`: production cluster overlays.
- `kubernetes/operators`: operator installation and configuration manifests.
- `kubernetes/crds`: custom resource definitions owned or consumed by SkyOps.
- `kubernetes/admission`: admission controller configuration and policy bindings.

Why it exists:

- Supports Kubernetes-native deployment workflows beyond Helm alone.
- Provides a clean home for CRDs, operators, overlays, and admission assets.
- Avoids mixing cluster manifests into service folders.

## 9. `terraform/` and `opentofu/`

`terraform/` and `opentofu/` contain infrastructure-as-code definitions for cloud resources.

Recommended folders:

- `terraform/modules`: reusable cloud infrastructure modules.
- `terraform/environments/dev`: dev environment composition.
- `terraform/environments/staging`: staging environment composition.
- `terraform/environments/production`: production environment composition.
- `terraform/providers`: provider version constraints and provider conventions.
- `terraform/backends`: backend configuration patterns and state isolation documentation.
- `opentofu/modules`: OpenTofu-compatible reusable modules.
- `opentofu/environments/*`: OpenTofu environment compositions.
- `opentofu/providers`: OpenTofu provider conventions.
- `opentofu/backends`: OpenTofu state backend patterns.

Why it exists:

- Supports both Terraform and OpenTofu requirements without hiding compatibility differences.
- Keeps reusable modules separate from environment instantiations.
- Enables strict state isolation by environment and production account.

## 10. `scripts/`

`scripts/` contains executable repository automation intended for humans and CI.

Recommended folders:

- `scripts/bootstrap`: first-time setup, dependency installation orchestration, and local environment bootstrap.
- `scripts/dev`: local development helpers.
- `scripts/ci`: CI entrypoints that wrap lower-level tools consistently.
- `scripts/release`: release preparation, changelog, tagging, and promotion helpers.
- `scripts/migrations`: migration orchestration helpers.
- `scripts/security`: local security scans, secret checks, and dependency audit helpers.
- `scripts/maintenance`: repository cleanup, generated file refresh, and operational maintenance scripts.

Why it exists:

- Keeps executable workflows versioned and easy to discover.
- Prevents CI configuration from becoming the only place where build behavior is documented.
- Allows local and CI workflows to share the same commands.

## 11. `docs/`

`docs/` contains durable human-readable product, architecture, operational, and governance documentation.

Recommended folders:

- `docs/architecture`: architecture blueprints, system diagrams, domain maps, and repository design.
- `docs/adr`: Architecture Decision Records.
- `docs/api`: API guidelines, generated API references, idempotency, pagination, and error standards.
- `docs/operations`: deployment, backup, restore, scaling, upgrade, and incident management documentation.
- `docs/security`: threat models, access control model, data classification, encryption, compliance, and audit guidance.
- `docs/ai`: AI governance, prompt lifecycle, RAG design, MCP policy, evaluation strategy, and model routing.
- `docs/runbooks`: step-by-step operational procedures.
- `docs/platform`: platform engineering practices and internal developer platform conventions.
- `docs/product`: product terminology, capability maps, and enterprise requirements.
- `docs/contributing`: contribution workflow, coding standards, review expectations, and local development guidance.

Why it exists:

- Makes architecture and operations reviewable with the same rigor as code.
- Gives future teams a durable record of decisions and operating models.
- Supports enterprise customer trust through strong security and operations documentation.

## 12. `tests/`

`tests/` contains cross-cutting and system-level tests that do not belong to one service or package.

Recommended folders:

- `tests/unit`: shared unit-test fixtures and repository-wide test conventions.
- `tests/integration`: multi-service integration scenarios.
- `tests/contract`: API and event compatibility tests.
- `tests/e2e`: full product workflows across apps, services, workers, and infrastructure dependencies.
- `tests/performance`: load, stress, soak, and capacity tests.
- `tests/security`: authorization, policy, dependency, image, and supply-chain security tests.
- `tests/chaos`: resilience experiments and failure-mode validation.
- `tests/migration`: database and infrastructure migration compatibility tests.
- `tests/fixtures`: reusable test fixtures, event samples, synthetic tenants, and sample repositories.

Why it exists:

- Makes cross-domain quality explicit.
- Prevents system-level tests from being hidden in individual services.
- Supports enterprise-grade regression, scalability, and reliability validation.

Placement rules:

- Service-local unit tests live beside the owning service.
- Cross-service tests live in `tests/`.
- Contract tests live in `tests/contract` and reference `packages/contracts`.

## 13. `examples/`

`examples/` contains safe, synthetic examples for users, developers, sales engineering, and integration testing.

Recommended folders:

- `examples/pipelines`: example CI/CD pipeline definitions.
- `examples/deployments`: example release and deployment workflows.
- `examples/infrastructure`: sample Terraform/OpenTofu stacks.
- `examples/kubernetes`: sample Kubernetes manifests and deployment targets.
- `examples/ai-workflows`: example AI-assisted operations and automation workflows.
- `examples/integrations`: sample provider integrations and webhook payloads.
- `examples/sdk-usage`: sample SDK and CLI usage patterns.

Why it exists:

- Provides realistic reference material without coupling examples to production services.
- Helps customers and internal teams understand intended platform usage.
- Supplies safe fixtures for demos and documentation.

## 14. `tools/`

`tools/` contains repository-owned developer and CI tooling.

Recommended folders:

- `tools/generators`: scaffolding for services, workers, packages, docs, ADRs, and contracts.
- `tools/linters`: custom lint rules and repository policy checks.
- `tools/codegen`: OpenAPI, AsyncAPI, Protobuf, SDK, and client generation orchestration.
- `tools/contract-testing`: compatibility check tooling for APIs and events.
- `tools/load-testing`: load-test runners and reusable performance scenarios.
- `tools/migration-tools`: database and infrastructure migration validation tools.
- `tools/release-tools`: release manifest, changelog, tag, artifact, and promotion tooling.
- `tools/local-dev`: local cluster, seed data, and developer productivity tools.

Why it exists:

- Keeps reusable engineering automation separate from product runtime code.
- Encourages generated consistency instead of manual repetition.
- Gives platform engineers a clear place to improve repository ergonomics.

## 15. `configs/`

`configs/` contains shared static configuration for repository tooling.

Recommended folders:

- `configs/eslint`: shared ESLint configuration.
- `configs/typescript`: shared TypeScript compiler configuration.
- `configs/prettier`: shared formatting rules.
- `configs/tailwind`: shared Tailwind CSS configuration presets.
- `configs/go`: Go linting, formatting, and static analysis configuration.
- `configs/python`: Python linting, formatting, typing, and packaging configuration.
- `configs/markdown`: markdown linting and documentation formatting configuration.
- `configs/commitlint`: commit message policy.
- `configs/renovate`: dependency update policy.
- `configs/security`: secret scanning, dependency scanning, and supply-chain configuration.
- `configs/telemetry`: repository-wide telemetry naming and instrumentation configuration.

Why it exists:

- Prevents every package and service from inventing its own tooling configuration.
- Enables centralized upgrades of repository standards.
- Makes CI behavior reproducible locally.

## 16. `.github/`

`.github/` contains GitHub-specific automation and governance.

Recommended folders and files:

- `.github/workflows`: CI, security scans, previews, releases, docs publishing, and scheduled maintenance workflows.
- `.github/actions`: composite GitHub Actions reused by workflows.
- `.github/ISSUE_TEMPLATE`: structured issue templates.
- `.github/PULL_REQUEST_TEMPLATE.md`: pull request checklist covering architecture, contracts, tests, migrations, security, and operations.
- `.github/CODEOWNERS`: ownership rules for services, packages, infrastructure, docs, and security-sensitive areas.

Why it exists:

- Keeps repository governance explicit.
- Enables path-based ownership and review requirements.
- Standardizes CI/CD workflows as the team grows.

## 17. AI prompt library location

The AI prompt library should live under `ai/prompts`.

Recommended folders:

- `ai/prompts/system`: product-wide system prompts and global AI behavior instructions.
- `ai/prompts/agents`: role-specific prompts for DevOps, SRE, security, platform engineering, release management, and support agents.
- `ai/prompts/workflows`: prompts tied to specific workflows such as incident triage, pipeline failure analysis, deployment review, and infrastructure drift explanation.
- `ai/prompts/evaluations`: prompts used for regression tests, golden answers, red-team tests, and offline scoring.
- `ai/prompts/guardrails`: refusal, escalation, approval, data handling, and tool-use policy prompts.

Why it exists:

- Keeps prompts versioned, reviewable, testable, and auditable.
- Separates AI product behavior from Python implementation details.
- Enables prompt evaluation and rollback independent of service source changes.

Related AI folders:

- `ai/rag`: RAG collection definitions, ingestion configuration, and retrieval policies.
- `ai/mcp`: MCP server registry, tool definitions, and tool execution policies.
- `ai/evals`: AI quality, safety, regression, and task success evaluations.

## 18. Shared package strategy

Shared packages should be categorized into four groups:

1. **Generated artifacts:** API clients, event models, schemas, and SDK surfaces generated from contracts.
2. **Cross-cutting primitives:** telemetry, errors, config, events, authz, feature flags, and test utilities.
3. **Runtime kits:** language-specific starter libraries for Go, Python, and TypeScript.
4. **Testing utilities:** fixtures, fake providers, contract test harnesses, and integration test scaffolding.

Rules:

- Shared packages must be small and stable.
- Shared packages must not contain domain workflows.
- Shared packages must publish explicit versions or change metadata even inside the monorepo.
- Shared packages must define owners and consumers.
- Generated packages must be reproducible from committed contracts.

Why this strategy exists:

- Prevents shared libraries from becoming a hidden monolith.
- Keeps domain ownership inside services while standardizing platform behavior.
- Allows internal SDKs and external SDKs to share generated foundations.

## 19. Dependency strategy

Recommended dependency approach:

- Use a workspace-aware package manager for TypeScript applications, packages, SDKs, and extension code.
- Use Go modules for Go services, workers, CLI, and Go SDKs.
- Use Python lockfiles for Python AI services, Python SDKs, and AI evaluation tooling.
- Pin Docker base images by digest for production images.
- Pin Terraform and OpenTofu providers and modules to explicit versions.
- Use automated dependency update pull requests with test, compatibility, and security checks.
- Generate SBOMs for every production image and released artifact.
- Maintain dependency ownership for critical libraries such as authentication, Kubernetes clients, OpenTelemetry, database drivers, AI providers, and serialization libraries.

Why it exists:

- Supports reproducible builds and auditable releases.
- Reduces security risk through automated updates and scanning.
- Allows different ecosystems to evolve without forcing one global dependency model.

## 20. Build strategy

Recommended build strategy:

- Build only affected projects in CI using dependency graph awareness.
- Require all generated files to be up to date before merge.
- Build frontend apps separately from backend services and AI services.
- Build service containers independently with standardized base images.
- Build SDKs from versioned contracts.
- Build the CLI as a separately released artifact.
- Build the VS Code extension as a separately versioned distributable.
- Build Helm charts and infrastructure modules through validation pipelines.
- Publish immutable artifacts with Git SHA, version, build timestamp, and provenance metadata.

Why it exists:

- Keeps CI fast as the monorepo grows.
- Supports independent deployment and release cadences.
- Produces artifacts that are traceable from production back to source, contracts, and commits.

## 21. Versioning strategy

Recommended versioning strategy:

- Product releases use semantic versioning: `MAJOR.MINOR.PATCH`.
- Services use image tags based on Git SHA and release version when included in a release train.
- APIs use explicit versioned paths and compatibility policies.
- Events use versioned schema names and versioned NATS subjects.
- SDKs use semantic versioning aligned to public API compatibility.
- CLI versions use semantic versioning and document supported platform API versions.
- Helm charts use chart versions and application versions separately.
- Terraform/OpenTofu modules use immutable version tags.
- Prompt libraries use prompt version metadata and changelog entries because prompt behavior affects production outcomes.

Why it exists:

- Allows services to deploy independently without breaking consumers.
- Makes customer-facing SDKs and CLI tools safe to adopt.
- Gives AI behavior the same rollback and audit discipline as code.

## 22. Environment configuration

Recommended environment configuration locations:

- `deployments/environments/local`: local defaults and service composition.
- `deployments/environments/dev`: shared dev configuration.
- `deployments/environments/staging`: production-like validation configuration.
- `deployments/environments/production`: production configuration bindings.
- `deployments/environments/preview`: ephemeral pull-request configuration.
- `deployments/environments/sandbox`: demo and trial configuration.
- `configs/`: shared static tooling configuration.
- `packages/config`: runtime configuration schema and validation conventions.
- `infrastructure/secrets`: secret reference and rotation architecture.

Rules:

- Non-secret configuration may be versioned when safe.
- Secret values must never be committed.
- Production configuration must be isolated from non-production configuration.
- Tenant-level settings belong in domain services, not in static files.
- AI model provider configuration must define provider, model, context policy, data retention, and tool permissions per environment.

Why it exists:

- Separates local, preview, staging, production, and tenant-specific concerns.
- Makes configuration reviewable without exposing secrets.
- Supports enterprise controls for region, isolation, data retention, and AI governance.

## 23. Naming conventions

Repository-level conventions:

- Use lowercase kebab-case for folders: `pipeline-service`, `ai-tool-gateway`, `vscode-extension`.
- Use singular domain names only when the product domain is singular by convention; otherwise prefer clear bounded-context names.
- Services end with `-service` unless they are engines, routers, or gateways with established architectural roles.
- Workers use action-oriented names: `pipeline-runner`, `drift-detector`, `ai-context-builder`.
- Packages use short capability names: `contracts`, `telemetry`, `authz`, `testkit`.

Runtime naming:

- Go module names should map to repository paths and service names.
- Python package names should use snake_case equivalents of folder names.
- TypeScript package names should use the `@skyops/` scope.
- Container images should use `skyops/<component-name>`.
- Helm releases should use `skyops-<component-name>`.

Why it exists:

- Keeps naming predictable across code, containers, Kubernetes, CI, dashboards, and documentation.
- Reduces cognitive load when new teams join the project.
- Makes ownership and deployment boundaries obvious.

## 24. Folder naming conventions

Folder naming standards:

- Use lowercase kebab-case for all product, service, worker, tool, and infrastructure folders.
- Use plural top-level category folders such as `apps`, `services`, `packages`, `docs`, `tests`, and `examples`.
- Use environment folder names exactly as `local`, `dev`, `staging`, `production`, `preview`, and `sandbox`.
- Use language folders only where they clarify runtime ownership, such as `services/go` and `services/python`.
- Avoid generic folders such as `misc`, `common`, `shared-stuff`, or `new`.
- Avoid deep nesting unless it reflects ownership, environment, or deployable boundaries.

Why it exists:

- Predictable folder names make automation and ownership rules easier.
- Consistent names help CI detect affected projects.
- Avoiding vague folders prevents architecture decay.

## 25. File naming conventions

File naming standards:

- Documentation files use lowercase kebab-case: `monorepo-structure.md`.
- ADR files use zero-padded sequence numbers and kebab-case titles: `0001-use-nats-for-events.md`.
- Configuration examples use `.example` suffixes when they represent templates.
- Environment-specific files include the environment name when they are not already inside an environment folder.
- Contract files include domain, resource, and version where applicable.
- Generated files must include a generated-file marker and should live under generated output folders where possible.
- Test fixture files should describe scenario and domain, such as `pipeline-run-failed.fixture.json`.
- Secret files must never contain real secret values and should be named as templates only.

Why it exists:

- Makes files searchable and understandable without opening them.
- Supports code generation and CI validation.
- Prevents accidental secret or generated-file drift.

## 26. Where every future module should live

Use this placement guide for future growth:

| Future module type | Location |
| --- | --- |
| Customer-facing web product surface | `apps/web` |
| Internal administration surface | `apps/admin` |
| Self-hosted operational UI | `apps/console` |
| Documentation website | `apps/docs-site` |
| Marketing website | `apps/marketing-site` |
| VS Code extension capability | `apps/vscode-extension` |
| New Go domain service | `services/go/<domain>-service` |
| New Python AI service | `services/python/<ai-domain>-service` |
| Async processor or controller | `workers/<purpose>` |
| External TypeScript SDK | `sdks/typescript` |
| External Go SDK | `sdks/go` |
| External Python SDK | `sdks/python` |
| CLI capability | `cli/skyops` |
| API or event contract | `packages/contracts` |
| Generated service client | `packages/clients` |
| Cross-cutting Go utility | `packages/go-kit` |
| Cross-cutting Python utility | `packages/python-kit` |
| Cross-cutting TypeScript utility | `packages/ts-kit` |
| AI prompt | `ai/prompts/<category>` |
| RAG collection or ingestion definition | `ai/rag` |
| MCP server or tool definition | `ai/mcp` |
| AI evaluation | `ai/evals` |
| Docker base image or Compose profile | `docker` |
| Kubernetes manifest, CRD, or overlay | `kubernetes` |
| Helm chart | `infrastructure/helm` |
| Terraform module or environment | `terraform` |
| OpenTofu module or environment | `opentofu` |
| GitOps app or cluster binding | `deployments/gitops` |
| Environment deployment values | `deployments/environments/<environment>` |
| Repository automation script | `scripts/<category>` |
| Developer tool or generator | `tools/<category>` |
| Shared static config | `configs/<tool-or-domain>` |
| Cross-service integration test | `tests/integration` |
| API or event contract test | `tests/contract` |
| End-to-end product test | `tests/e2e` |
| Performance or load test | `tests/performance` |
| Security validation | `tests/security` |
| Chaos or resilience test | `tests/chaos` |
| Migration compatibility test | `tests/migration` |
| User or customer example | `examples/<domain>` |
| Architecture documentation | `docs/architecture` |
| Architecture decision record | `docs/adr` |
| Operational runbook | `docs/runbooks` |
| Security documentation | `docs/security` |
| AI governance documentation | `docs/ai` |

Why it exists:

- Gives future contributors a deterministic placement model.
- Prevents monorepo sprawl as new products, services, integrations, and AI capabilities are added.
- Keeps ownership, deployment, testing, and documentation aligned with the architecture.
