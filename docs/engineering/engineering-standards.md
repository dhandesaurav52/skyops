# SkyOps Engineering Standards

## Purpose

This document is the official engineering handbook for SkyOps. It defines the mandatory standards that every contributor, maintainer, reviewer, and AI coding agent must follow when changing the SkyOps codebase.

SkyOps is expected to become a production-grade enterprise SaaS platform. Engineering decisions must therefore optimize for correctness, security, operability, maintainability, traceability, and long-term product velocity. These standards do not replace the approved Software Architecture, Monorepo Structure, or Product Vision. They translate those approved documents into day-to-day engineering rules.

## Scope

These standards apply to all code, configuration, documentation, infrastructure definitions, tests, generated artifacts, AI prompts, and automation in the SkyOps monorepo.

Contributors must not use this document to redesign existing architecture, APIs, databases, or UI. When a standard conflicts with an approved architecture decision record, the ADR controls until it is superseded by a newer accepted ADR.

## 1. Engineering Principles

1. **Production first:** Treat every change as if it may eventually run in a regulated enterprise production environment. Avoid shortcuts that compromise reliability, security, data integrity, auditability, or maintainability.
2. **Architecture aligned:** Follow the approved domain-oriented, API-first, event-driven, cloud-native architecture. Do not introduce cross-domain coupling or hidden shared business logic.
3. **Security by default:** Secure behavior must be the default path. Authentication, authorization, tenant isolation, input validation, secret handling, and auditability are mandatory design concerns.
4. **Operational excellence:** Code must be observable, diagnosable, measurable, and safe to operate. Every service and worker must support structured logs, metrics, traces, health checks, graceful shutdown, and clear failure modes.
5. **Small, reversible changes:** Prefer incremental changes that are easy to review, test, deploy, monitor, and roll back.
6. **Explicit contracts:** Service boundaries, package APIs, events, configuration, and error types must be explicit and versioned where external consumers depend on them.
7. **Tests are part of the feature:** A change is incomplete without appropriate automated tests or a documented reason why no test applies.
8. **Documentation is part of the product:** Architecture, operations, security, AI behavior, and contributor-facing rules must be documented with the same care as code.
9. **Human and AI contributors follow the same bar:** AI-generated code must satisfy all standards, pass review, include tests, and remain understandable to human maintainers.

## 2. Clean Architecture Guidelines

SkyOps code must separate business policy from delivery mechanisms.

- Domain rules must not depend on HTTP frameworks, database clients, queue clients, UI frameworks, CLI libraries, or cloud SDKs.
- Use clear boundaries between domain logic, application orchestration, adapters, infrastructure integrations, and presentation layers.
- Dependencies must point inward toward stable business concepts. Outer layers may know about inner layers; inner layers must not know about outer-layer frameworks.
- Keep use cases explicit. A use case should describe one application action, coordinate dependencies through interfaces, enforce policy, and return domain-safe results.
- Repositories, clients, publishers, identity providers, storage implementations, and external integrations must be adapters behind interfaces owned by the consuming domain.
- Shared packages may provide primitives, generated clients, telemetry, errors, configuration, authentication helpers, authorization helpers, and test utilities. Shared packages must not own product-specific workflow decisions.
- Cross-domain communication must happen through approved contracts, service APIs, events, or documented integration points, not direct database access or import chains into another domain's internals.

## 3. SOLID Principles

- **Single Responsibility:** A module, type, function, component, or package should have one clear reason to change. Split code when responsibilities mix domain rules, transport, persistence, formatting, and external calls.
- **Open/Closed:** Extend behavior through composition, strategies, interfaces, configuration, or registered handlers rather than editing unrelated code paths.
- **Liskov Substitution:** Interface implementations must preserve the promised behavior, error semantics, security checks, and tenant boundaries.
- **Interface Segregation:** Prefer small, consumer-owned interfaces. Do not create large service interfaces that force implementations to include unused methods.
- **Dependency Inversion:** High-level domain and application logic should depend on abstractions, not concrete frameworks, SDKs, or infrastructure clients.

SOLID must not be used as an excuse for needless abstraction. Apply it to improve clarity, testability, and change isolation.

## 4. DRY Principles

Do not duplicate business logic, security rules, validation rules, event schemas, configuration parsing, or integration behavior.

Acceptable duplication includes small, obvious code that avoids premature abstraction, test setup that improves readability, and isolated examples in documentation. Unacceptable duplication includes copied authorization checks, copied tenant scoping, copied API response shaping, copied retry logic, copied environment parsing, and copied AI safety rules.

Before adding a helper or shared package, verify that the abstraction is stable, named clearly, and not hiding domain ownership.

## 5. KISS Principles

Prefer the simplest design that satisfies current approved requirements while preserving safety and maintainability.

- Use straightforward control flow.
- Keep functions short and readable.
- Prefer explicit names over clever patterns.
- Avoid magic, reflection, metaprogramming, global state, and implicit side effects unless clearly justified.
- Choose boring, proven technology for production paths.
- Optimize for the next maintainer's understanding, not for novelty.

## 6. YAGNI Guidelines

Do not build capabilities that are not required by approved product, architecture, or ADR decisions.

- Do not add unused extension points, speculative generic frameworks, placeholder services, unused configuration flags, or fake future integrations.
- Do not create abstractions for a single implementation unless needed for testability, boundary control, or dependency inversion.
- Do not add background jobs, caches, queues, or distributed coordination without a concrete requirement.
- Document deferred ideas in an issue or ADR proposal instead of shipping unused code.

## 7. Monorepo Development Rules

- Respect the approved monorepo structure and domain ownership boundaries.
- Deployable applications belong in `apps/`, backend services in `services/`, asynchronous processors in `workers/`, reusable packages in `packages/`, SDKs in `sdks/`, infrastructure assets in their approved infrastructure folders, and documentation in `docs/`.
- Do not introduce new root-level directories without an ADR or maintainer approval.
- Keep package dependencies acyclic.
- Shared packages must remain small, stable, documented, and well tested.
- Generated artifacts must be reproducible from committed sources and generated by documented commands.
- Each service, worker, package, or app must own its tests near the code or in the approved test hierarchy.
- Local development commands should be documented and automatable.

## 8. Folder Naming Standards

- Use lowercase kebab-case for folders: `pipeline-service`, `ai-tool-gateway`, `feature-flags`.
- Folder names must describe ownership or purpose, not implementation trivia.
- Avoid abbreviations unless they are standard domain terms such as `api`, `cli`, `sdk`, `sso`, `scim`, `slo`, `rbac`, or `mcp`.
- Keep domain folders singular or compound according to the approved monorepo structure.
- Do not use spaces, uppercase letters, dates, personal names, or temporary labels in folder names.

## 9. File Naming Standards

- Use lowercase kebab-case for documentation, configuration, scripts, and web assets.
- Use language-standard naming for source files:
  - Go files use lowercase snake_case where appropriate, such as `pipeline_run.go`.
  - TypeScript files use kebab-case for modules and PascalCase only for framework component files when the framework convention requires it.
  - Python files use lowercase snake_case.
- Test files must use language conventions: `_test.go`, `.test.ts`, `.spec.ts`, or `test_*.py`.
- Avoid vague names such as `utils`, `helpers`, `common`, `misc`, or `temp`. If such a file is unavoidable, its scope must be narrow and documented.

## 10. Package Naming Standards

- Package names must communicate purpose and ownership.
- Go package names must be short, lowercase, singular where idiomatic, and free of underscores.
- TypeScript package names must use the approved workspace namespace once established, such as `@skyops/contracts` or `@skyops/telemetry`.
- Python packages must use lowercase snake_case and avoid collisions with standard library modules.
- Do not create catch-all packages for business logic.
- Package public APIs must be documented, versioned when consumed externally, and tested.

## 11. Go Coding Standards

- Use idiomatic Go with `gofmt`, `go vet`, and static analysis in CI.
- Keep packages cohesive and small.
- Pass `context.Context` as the first parameter for request-scoped, I/O, or cancellable operations.
- Return errors explicitly; do not panic in normal application flow.
- Wrap errors with meaningful context while preserving machine-checkable error classification where needed.
- Use interfaces at consumer boundaries, not as premature provider-side abstractions.
- Document exported types, functions, constants, and packages.
- Avoid global mutable state.
- Use table-driven tests for validation matrices and edge cases.
- Ensure goroutines have cancellation, lifecycle ownership, bounded concurrency, and error reporting.

## 12. TypeScript Coding Standards

- Use strict TypeScript. Do not disable type checking to bypass design issues.
- Prefer explicit domain types over unstructured objects.
- Avoid `any`; when unavoidable, isolate it at integration boundaries and validate immediately.
- Use schema validation for external input, API responses, environment variables, and persisted serialized data.
- Keep UI state, API clients, domain models, and presentation components separated.
- Export only intentional public APIs from packages.
- Prefer named exports for shared packages.
- Handle asynchronous operations explicitly and avoid floating promises.
- Use formatting, linting, and type checking in CI.

## 13. Python Coding Standards

- Use modern Python with type hints for public functions, service boundaries, and AI workflow interfaces.
- Use formatters, linters, and static type checking appropriate to the package.
- Keep AI, ML, retrieval, and integration code modular and testable.
- Validate external inputs with explicit schemas or models.
- Do not hide failures with broad exception handlers.
- Avoid mutable default arguments.
- Keep side effects out of module import time.
- Document public modules, classes, and functions.
- Pin production dependencies through approved dependency management.

## 14. API Coding Standards

- SkyOps is API-first. Contracts must be designed, reviewed, documented, and versioned before consumers depend on them.
- API handlers must be thin. They should authenticate, authorize, validate, call application use cases, map errors, and return responses.
- Every API request must enforce tenant context and authorization.
- Use consistent request IDs, correlation IDs, pagination, filtering, sorting, idempotency, and error response formats.
- Never expose internal stack traces, raw database errors, secrets, tokens, or provider credentials in API responses.
- Breaking API changes require versioning, migration guidance, and explicit approval.
- Public API behavior must have contract tests.

## 15. Error Handling Standards

- Errors must be actionable, classified, and safe to expose at the appropriate boundary.
- Distinguish validation errors, authentication errors, authorization errors, not-found errors, conflict errors, rate-limit errors, dependency errors, and internal errors.
- Preserve root causes for logs and traces while returning sanitized messages to users.
- Do not swallow errors.
- Do not retry non-retryable failures.
- Use timeouts, cancellation, and circuit-breaking for external dependencies.
- Error messages must not contain secrets, tokens, credentials, or sensitive tenant data.

## 16. Logging Standards

- Use structured logging in all services, workers, CLIs, and automation where practical.
- Include request ID, correlation ID, tenant identifiers when safe, service name, operation name, and error classification.
- Do not log secrets, credentials, tokens, private keys, raw authorization headers, personal data beyond approved policy, or sensitive customer payloads.
- Use appropriate levels: debug for development detail, info for lifecycle events, warn for recoverable anomalies, error for failed operations requiring attention.
- Logs must support incident response and audit reconstruction without becoming a data leak.

## 17. Configuration Standards

- Configuration must be explicit, typed, validated at startup, and documented.
- Required configuration must fail fast with clear messages.
- Defaults must be safe for local development and never unsafe for production.
- Runtime behavior must not depend on hidden global configuration.
- Configuration should be layered predictably: defaults, environment-specific files where approved, environment variables, and secret providers.
- Configuration changes that affect production behavior require tests and release notes where applicable.

## 18. Environment Variable Standards

- Environment variables must use uppercase snake case and the `SKYOPS_` prefix for SkyOps-owned settings.
- Names must be descriptive: `SKYOPS_API_BASE_URL`, `SKYOPS_LOG_LEVEL`, `SKYOPS_AUTH_ISSUER_URL`.
- Do not store secrets directly in committed files.
- Every environment variable must be documented with purpose, type, required status, default, and allowed values.
- Validate environment variables at startup and fail fast on invalid values.
- Avoid boolean flags with ambiguous names; prefer explicit positive names.

## 19. Security Standards

- Follow secure-by-default design for every change.
- Validate and sanitize all external input.
- Enforce tenant isolation in application logic, persistence access, events, caches, logs, metrics, and AI context.
- Use least privilege for users, services, workers, CI jobs, cloud roles, and Kubernetes identities.
- Never hardcode secrets, credentials, tokens, private keys, or customer data.
- Use approved secret managers and short-lived credentials where possible.
- Protect against injection, SSRF, XSS, CSRF, insecure deserialization, path traversal, confused deputy flows, and dependency supply-chain risks.
- Security-sensitive changes require focused review.
- All security-relevant actions must be auditable.

## 20. Authentication Standards

- Authentication must be enforced for every protected API, UI route, worker action endpoint, and operational tool.
- Do not bypass authentication for convenience outside explicitly isolated local development flows.
- Support enterprise identity patterns including SSO, service accounts, API tokens, and short-lived credentials as approved architecture requires.
- Session and token handling must use secure storage, expiration, rotation, revocation, and audience validation.
- Authentication failures must be logged safely and must not reveal whether sensitive identifiers exist.

## 21. Authorization Standards

- Authorization must be checked after authentication and before accessing or mutating protected resources.
- Enforce organization, workspace, project, environment, and resource boundaries consistently.
- Do not rely only on UI hiding, route naming, client-side checks, or obscurity.
- Authorization decisions must be centralized through approved policy helpers or services where available.
- Privileged operations require explicit permission checks, audit events, and where applicable approval workflows.
- AI agents must be authorized as the user or service identity they act on behalf of; they must never receive elevated implicit permissions.

## 22. Git Workflow

- Work on feature branches, not directly on protected mainline branches.
- Keep commits focused and reviewable.
- Rebase or merge from the target branch regularly according to repository policy.
- Do not commit generated noise, local environment files, secrets, dependency caches, or unrelated formatting churn.
- Run relevant tests and checks before opening a pull request.
- Every merged change must be traceable to a pull request, review, CI result, and release note when applicable.

## 23. Branch Naming Convention

Use lowercase kebab-case with a type prefix:

- `feature/<short-description>`
- `fix/<short-description>`
- `docs/<short-description>`
- `refactor/<short-description>`
- `test/<short-description>`
- `chore/<short-description>`
- `security/<short-description>`

Examples: `docs/engineering-standards`, `fix/tenant-scope-validation`, `feature/pipeline-run-summary`.

## 24. Commit Message Convention

SkyOps uses Conventional Commits.

Format:

```text
<type>(optional-scope): <short imperative summary>

[optional body]

[optional footers]
```

Allowed common types include `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `build`, `ci`, `perf`, `security`, and `revert`.

Rules:

- Use imperative mood: `docs: add engineering standards`.
- Keep the summary concise and specific.
- Use scopes for services, packages, or documentation areas when helpful.
- Mark breaking changes with `BREAKING CHANGE:` in the footer.
- Do not mix unrelated changes in one commit.

## 25. Pull Request Standards

Every pull request must include:

- Clear title using Conventional Commit style where practical.
- Summary of what changed and why.
- Link to relevant issue, ticket, or ADR when applicable.
- Testing performed with exact commands.
- Screenshots or recordings for perceptible UI changes.
- Security, migration, deployment, and rollback notes when applicable.
- Documentation updates for user-visible, operator-visible, API, or contributor-facing changes.

Pull requests must be small enough for effective review. Large changes should be split by domain, layer, or migration stage.

## 26. Code Review Checklist

Reviewers must verify:

- The change follows approved architecture and monorepo boundaries.
- Business logic is not duplicated.
- Authentication, authorization, tenant isolation, and auditability are correct.
- Input validation and error handling are complete.
- Logs, metrics, traces, and operational behavior are adequate.
- Tests cover expected behavior, failure paths, edge cases, and security-sensitive paths.
- Naming, package structure, and dependency direction are appropriate.
- No secrets, sensitive data, or unrelated files are committed.
- Documentation is updated.
- AI-generated code, if present, is understandable, necessary, tested, and standards-compliant.

## 27. Documentation Standards

- Documentation must be accurate, versioned, and reviewed with related code changes.
- Use Markdown for repository documentation.
- Prefer clear headings, concise explanations, examples, and decision rationale.
- Architecture changes require ADRs.
- Operational behavior requires runbooks.
- Public APIs require reference documentation and examples.
- Security-sensitive behavior requires security documentation.
- AI prompts, tools, policies, and evaluations must be documented where they affect product behavior.

## 28. Testing Strategy

SkyOps uses layered testing:

- Unit tests verify isolated logic quickly.
- Integration tests verify interactions with real or realistic dependencies.
- Contract tests verify APIs, events, generated clients, and provider boundaries.
- End-to-end tests verify critical user and operational workflows.
- Performance tests verify latency, throughput, scalability, and resource usage.
- Security tests verify access controls, input validation, dependency vulnerabilities, and policy enforcement.

CI must run the fastest reliable checks early and reserve expensive suites for appropriate gates.

## 29. Unit Testing Standards

- Unit tests must be deterministic, fast, isolated, and readable.
- Test business rules, validation, error classification, authorization decisions, and edge cases.
- Avoid relying on networks, real databases, current time, random values, or global state unless controlled by test fixtures.
- Use table-driven tests where they improve coverage clarity.
- Each bug fix should include a regression test whenever practical.

## 30. Integration Testing Standards

- Integration tests must use realistic dependencies through approved local containers, test fixtures, or sandbox services.
- Verify persistence behavior, transactions, migrations, event publishing, external provider adapters, and service-to-service contracts.
- Tests must create and clean up their own data.
- Integration tests must not depend on production credentials or shared mutable cloud resources.
- Clearly mark tests that require optional external services.

## 31. End-to-End Testing Standards

- End-to-end tests must cover critical SkyOps workflows such as authentication, repository connection, pipeline execution, deployment traceability, policy enforcement, and operational visibility when those features exist.
- E2E tests should focus on high-value user journeys, not exhaustive UI permutations.
- Tests must be stable, observable, and runnable in CI.
- Use seeded test data and isolated tenants.
- Failures should provide actionable diagnostics, screenshots, traces, or logs where applicable.

## 32. Performance Standards

- Define performance expectations for latency, throughput, concurrency, resource usage, and queue processing where relevant.
- Avoid unbounded queries, unbounded fan-out, unbounded goroutines, uncontrolled recursion, and memory-heavy payload processing.
- Use pagination and streaming for large datasets.
- Cache only with explicit invalidation, tenant isolation, and observability.
- Performance-sensitive changes require benchmarks, load tests, or documented analysis.
- Degradation must be observable through metrics and alerts.

## 33. Accessibility Standards

- User-facing applications must meet WCAG 2.2 AA unless an approved exception exists.
- Build semantic, keyboard-accessible interfaces.
- Maintain sufficient color contrast.
- Provide labels, focus states, accessible names, and error descriptions.
- Do not rely on color alone to communicate state.
- Include accessibility checks in review and testing for UI changes.

## 34. Dependency Management Policy

- Add dependencies only when they provide clear value over standard library or existing approved packages.
- Prefer mature, maintained, secure, well-licensed libraries.
- Review transitive dependency impact.
- Pin versions through the approved package manager and lockfiles.
- Keep dependencies updated through automated tooling and reviewed upgrade PRs.
- Remove unused dependencies promptly.
- Do not introduce dependencies with incompatible licenses or unclear provenance.
- Security patches must be prioritized according to severity and exploitability.

## 35. Definition of Done

A change is done only when:

- It satisfies the approved requirement without unrelated scope expansion.
- It follows architecture, security, naming, testing, and documentation standards.
- Relevant automated tests pass.
- New behavior is observable and operable.
- Public or contributor-facing documentation is updated.
- API, event, configuration, migration, and release impacts are documented.
- Security and tenant isolation implications have been reviewed.
- The pull request has been reviewed and approved by appropriate owners.
- CI passes or failures are explicitly accepted by maintainers for documented environmental reasons.

## 36. AI Coding Rules

AI-generated code is allowed only when it follows all SkyOps standards. The human or agent submitting the change is accountable for the result.

Mandatory AI rules:

- Never duplicate business logic.
- Always write or update tests for behavior changes.
- Always document exported functions, types, packages, and public interfaces.
- Never hardcode secrets, credentials, tokens, endpoints containing secrets, or customer data.
- Never bypass authentication, authorization, tenant isolation, policy checks, audit logging, or approval workflows.
- Never create unnecessary abstractions, speculative frameworks, or unused extension points.
- Keep functions small, readable, and purpose-specific.
- Follow existing architecture, package boundaries, naming, and dependency direction.
- Reuse shared packages and generated contracts whenever appropriate.
- Do not invent APIs, schemas, infrastructure, or UI patterns that conflict with approved documents.
- Do not silence linters, type checkers, tests, or security tools without explicit justification.
- Do not add broad exception handlers or panic recovery to hide failures.
- Do not modify generated files manually unless the generation process explicitly requires it.
- Do not introduce new dependencies without explaining why existing code cannot satisfy the need.
- Make all assumptions explicit in PR notes when requirements are ambiguous.
- Prefer minimal diffs that preserve human maintainability.

AI output must be reviewed as carefully as human-written code. Generated code that is not understood by the contributor must not be committed.

## 37. AI Prompting Rules

Prompts used to generate SkyOps code, documentation, tests, or operational actions must direct the AI to:

- Follow the approved Product Vision, Software Architecture, Monorepo Structure, and Engineering Standards.
- Preserve domain boundaries and clean architecture.
- Avoid implementation beyond the requested scope.
- Include tests and documentation for behavior changes.
- Treat security, tenant isolation, authentication, authorization, auditability, and secret handling as mandatory.
- Reuse existing packages and patterns before creating new ones.
- Explain assumptions, trade-offs, and risks.
- Avoid fabricating facts about the repository; inspect actual files instead.
- Avoid destructive actions unless explicitly requested and reviewed.

AI prompts that can affect production, infrastructure, customer data, security posture, or privileged operations must include safety constraints and approval requirements.

## 38. Architecture Decision Record Process

Use ADRs for significant decisions that affect architecture, domain boundaries, technology choices, data ownership, deployment topology, security model, AI behavior, or long-term maintainability.

Each ADR must include:

- Title and status.
- Date.
- Context and problem statement.
- Decision.
- Alternatives considered.
- Consequences and trade-offs.
- Security, operational, and migration considerations where relevant.

ADR statuses include `proposed`, `accepted`, `superseded`, and `rejected`. Accepted ADRs remain binding until superseded by a newer accepted ADR.

## 39. Release Process

- Releases must be traceable from source commits to artifacts, deployments, changelogs, approvals, and runtime health.
- Release candidates must pass required CI, security checks, and environment-specific gates.
- Production releases require rollback plans and monitoring expectations.
- Database, infrastructure, and contract migrations must be backward compatible unless a breaking-change process is approved.
- Release notes must identify user-visible changes, operational changes, security fixes, migrations, and known risks.
- Failed releases must produce an incident, postmortem, or follow-up issue when impact warrants it.

## 40. Versioning Strategy

- Public APIs, SDKs, CLI behavior, event contracts, and externally consumed packages must use semantic versioning.
- Increment the major version for breaking changes, minor version for backward-compatible features, and patch version for backward-compatible fixes.
- Internal services may version independently when deployment and compatibility boundaries require it.
- Event schemas and API contracts must support backward-compatible evolution whenever possible.
- Deprecations must include migration guidance, timelines, and observability for usage where practical.
- Build artifacts must be immutable and traceable to commit SHA, version, build metadata, and provenance.

## Enforcement

Maintainers, reviewers, CI systems, security tooling, and AI agents are responsible for enforcing these standards. Exceptions must be rare, documented, time-bound, and approved by the appropriate owners. Standards should evolve through pull requests and ADRs as SkyOps matures, but contributors must follow the current accepted version at all times.
