# SkyOps AI Architecture

## Purpose

This document is the official AI architecture blueprint for SkyOps. It defines how AI features, AI services, AI agents, model providers, knowledge retrieval, memory, tools, monitoring, and governance fit into the approved SkyOps product, software, backend, frontend, and security architecture.

SkyOps AI must help platform teams understand, operate, secure, and improve complex DevOps systems without weakening tenant isolation, human accountability, or production safety.

## Part 1 — AI philosophy

### 1. AI vision

SkyOps AI is an enterprise platform engineering copilot and operations intelligence layer. Its purpose is to convert fragmented operational data into explainable insights, safer decisions, faster investigations, and governed automation.

AI should support:

- Understanding projects, repositories, pipelines, deployments, Kubernetes resources, incidents, logs, metrics, traces, security findings, topology relationships, and documentation.
- Summarizing operational state in plain language with evidence.
- Recommending safe next actions with risk and blast-radius context.
- Assisting engineers through CLI, VS Code Extension, web UI, Topology Graph, notifications, and incident workflows.
- Executing approved tools only within explicit permissions, policy, and audit controls.

AI must not become an opaque production operator. It is an assistant, investigator, planner, reviewer, and optional executor under human and policy control.

### 2. AI design principles

1. **Tenant-bound intelligence:** AI context, retrieval, memory, embeddings, prompts, outputs, and tool execution are scoped to the requesting tenant and permissions.
2. **Evidence-first responses:** AI claims about systems must cite the underlying signals, logs, metrics, events, commits, manifests, or policy results where available.
3. **Human accountability:** High-impact actions require human review, approvals, and audit trails.
4. **Least-privilege tools:** Agents receive only the tools and data needed for the current task.
5. **Provider portability:** SkyOps supports multiple model providers through a routing layer and avoids coupling core product behavior to one vendor.
6. **Deterministic control plane:** AI may recommend or initiate workflows, but authoritative state transitions remain in SkyOps services and policies.
7. **Observable intelligence:** AI latency, cost, quality, safety events, retrieval quality, tool calls, and failure modes are measured.
8. **Graceful degradation:** AI features degrade when providers, indexes, memory, or tools are unavailable without blocking core platform operations.

### 3. Responsible AI guidelines

Responsible AI requirements:

- Clearly label AI-generated content.
- Distinguish facts, inferences, recommendations, uncertainty, and actions.
- Avoid claiming certainty when evidence is incomplete.
- Provide citations or evidence links for operational conclusions.
- Avoid leaking secrets, personal data, tenant data, or restricted operational data.
- Avoid manipulative, unsafe, or policy-bypassing instructions.
- Maintain auditability for AI-assisted decisions.
- Respect customer configuration for data retention, model providers, AI enablement, and sensitive data handling.

### 4. AI governance

AI governance defines who can create, change, deploy, evaluate, and operate AI capabilities.

Governed assets:

- Prompt templates and versions.
- Agent definitions and capabilities.
- Tool definitions and risk classifications.
- Model registry entries.
- Routing policies.
- Retrieval indexes and knowledge sources.
- Memory policies.
- Evaluation datasets and scorecards.
- Safety policies and approval requirements.

Governance controls:

- Owner for every prompt, agent, tool, model route, and knowledge source.
- Versioning and changelog for prompts and agent behavior.
- Review gates for production AI changes.
- Offline and online evaluation before rollout.
- Feature flags for staged deployment and rollback.
- Audit logs for prompt changes, model routing changes, tool executions, and AI-generated action proposals.

### 5. Explainability

Explainability is required for enterprise trust.

AI responses should expose:

- The question or task interpreted by the system.
- Relevant tenant, project, environment, and time range.
- Sources used for retrieval.
- Evidence links and timestamps.
- Confidence or uncertainty indicators.
- Assumptions and missing data.
- Why a recommendation is safe or risky.
- What tool actions were proposed or executed.

Explainability is especially important for incident response, security findings, production deployments, topology insights, and autonomous remediation proposals.

### 6. Human-in-the-loop strategy

Human-in-the-loop controls are based on action risk.

- **Read-only insights:** AI may answer directly if data access is authorized.
- **Low-risk suggestions:** AI may recommend steps and generate drafts for human use.
- **Medium-risk actions:** AI may prepare a proposed action requiring explicit user confirmation.
- **High-risk actions:** AI requires permission checks, policy checks, approval workflow, and possibly step-up MFA.
- **Restricted actions:** AI may explain and prepare evidence but cannot execute without an approved break-glass or formal workflow.

Examples of high-risk actions include production rollbacks, cluster changes, security policy changes, role assignments, API key creation, destructive infrastructure changes, and broad data export.

## Part 2 — AI platform architecture

### Overview

The SkyOps AI platform is composed of modular layers:

1. AI Gateway.
2. AI Service Layer.
3. Prompt Management.
4. Context Management.
5. Memory Management.
6. Knowledge Layer.
7. Embedding Layer.
8. Vector Search Layer.
9. Tool Execution Layer.
10. Agent Runtime.
11. Model Routing.
12. Model Registry.
13. AI Monitoring.
14. AI Security.

Each layer has clear responsibilities and trust boundaries.

### AI Gateway

The AI Gateway is the single controlled entry point for AI requests from the web app, CLI, VS Code Extension, notifications, incident workflows, and internal services.

Responsibilities:

- Authenticate caller context through approved platform mechanisms.
- Resolve tenant, workspace, project, environment, and user permissions.
- Enforce AI feature flags, entitlements, quotas, rate limits, and organization AI policy.
- Classify request intent and risk.
- Route requests to the appropriate AI service or agent runtime.
- Attach request IDs, trace IDs, and audit context.
- Normalize AI response envelopes without exposing provider details unnecessarily.

The AI Gateway does not directly mutate product resources. Mutations go through governed tool execution and domain services.

### AI Service Layer

The AI Service Layer contains domain-aligned AI capabilities such as summarization, investigation, recommendation, code review assistance, topology insights, incident analysis, documentation generation, and cost analysis.

Responsibilities:

- Orchestrate prompts, retrieval, memory, model calls, and tools.
- Enforce task-specific policies.
- Format explainable outputs.
- Handle retries, fallback, and partial results.
- Emit AI telemetry and audit events.

Services should be modular and should not embed provider-specific logic directly.

### Prompt Management

Prompt Management owns prompt templates, variables, versions, lifecycle, tests, approval state, and rollout policy. Details are defined in `docs/ai/prompt-framework.md`.

Responsibilities:

- Store approved prompt templates and metadata.
- Resolve prompt versions by feature, agent, tenant policy, and experiment.
- Validate variables and required context.
- Prevent unsafe prompt modifications from bypassing review.
- Support evaluations and rollback.

### Context Management

Context Management determines what information is relevant and allowed for a request.

Responsibilities:

- Resolve explicit user context such as selected project, resource, topology node, incident, or time range.
- Retrieve authorized domain context from services.
- Apply tenant and permission filters.
- Rank context by relevance, freshness, and reliability.
- Fit context into model constraints.
- Track context sources for explainability.

Context Management must avoid over-fetching sensitive data and must never use unauthorized data to improve answers.

### Memory Management

Memory Management handles session, conversation, project, organization, and long-term memory. Details are defined in `docs/ai/memory-system.md`.

Responsibilities:

- Store and retrieve permitted memory by scope.
- Enforce retention and expiration policies.
- Index memory for retrieval.
- Redact sensitive data.
- Separate tenant memory.
- Support user and organization controls.

### Knowledge Layer

The Knowledge Layer contains authoritative and derived knowledge sources available to AI.

Sources:

- Product documentation.
- Runbooks.
- Architecture documents.
- Repository metadata.
- Deployment history.
- Pipeline runs.
- Kubernetes resource state.
- Logs, metrics, traces, alerts, and incidents.
- Security findings and policies.
- Topology graph relationships.
- Audit events where permitted.

Knowledge sources must define owner, freshness, access policy, indexing strategy, retention, and citation behavior.

### Embedding Layer

The Embedding Layer converts approved text or structured summaries into vector representations.

Responsibilities:

- Select embedding models through the model registry.
- Normalize and chunk source content.
- Preserve tenant, source, permission, and freshness metadata.
- Redact or exclude sensitive data based on policy.
- Re-embed content when source data or embedding model versions change.
- Track embedding lineage for evaluation and deletion.

### Vector Search Layer

The Vector Search Layer retrieves semantically relevant context.

Responsibilities:

- Enforce tenant and permission filters before returning results.
- Support hybrid retrieval combining vector, keyword, metadata, topology, and time filters.
- Return source metadata for explainability.
- Apply freshness and reliability scoring.
- Avoid cross-tenant indexes unless isolation is provably enforced and monitored.
- Support deletion and reindexing for lifecycle events.

### Tool Execution Layer

The Tool Execution Layer is the only path for AI-initiated actions.

Responsibilities:

- Register available tools with owner, purpose, input schema, output schema, risk level, required permissions, approval requirements, and audit behavior.
- Validate tool arguments.
- Re-authorize the requesting actor and delegated agent.
- Enforce policy, environment protections, and rate limits.
- Require confirmation or approval for risky actions.
- Execute through domain services rather than direct infrastructure access.
- Record tool proposals, approvals, executions, outcomes, and errors.

### Agent Runtime

The Agent Runtime manages specialized agents, task planning, tool access, intermediate reasoning controls, memory access, and execution lifecycle.

Responsibilities:

- Select agent based on task intent.
- Provide least-privilege tools and context.
- Coordinate multi-step investigations.
- Stop when confidence is low, policy blocks action, or human approval is required.
- Produce structured outputs for UI, CLI, notifications, and audit.

Agent design is detailed in `docs/ai/agent-architecture.md`.

### Model Routing

Model Routing selects the best model for each task based on capability, policy, latency, cost, availability, data sensitivity, context window, and output requirements. Details are defined in `docs/ai/llm-orchestration.md`.

### Model Registry

The Model Registry records approved model providers and model capabilities.

Registry metadata:

- Provider and model name.
- Supported modalities.
- Context window.
- Cost profile.
- Latency profile.
- Data handling policy.
- Region availability.
- Tool/function support.
- Streaming support.
- Safety capabilities.
- Approved use cases.
- Restricted use cases.
- Evaluation results.
- Retirement date where applicable.

### AI Monitoring

AI Monitoring measures reliability, safety, quality, and cost.

Signals:

- Request volume, latency, streaming duration, and errors.
- Token usage and cost by tenant, feature, model, and agent.
- Retrieval hit rate and citation coverage.
- Tool proposal and execution outcomes.
- Approval rates and rejection reasons.
- Hallucination reports and correction events.
- Prompt injection detection events.
- Policy denials and permission denials.
- User feedback and quality scores.

### AI Security

AI Security applies the approved security architecture to all AI workflows.

Controls:

- Tenant isolation.
- Permission-filtered retrieval.
- Prompt injection defense.
- Sensitive information redaction.
- Tool execution authorization.
- Model provider policy enforcement.
- Audit logging.
- Rate limiting and quota management.
- Data retention controls.
- Human approval for high-risk actions.

## Part 7 — AI safety summary

AI safety is embedded throughout the platform.

### Hallucination mitigation

- Ground responses in retrieved evidence.
- Ask for clarification when context is insufficient.
- Use confidence indicators.
- Prefer summaries of known platform data over unsupported claims.
- Evaluate prompts and agents against known-answer datasets.

### Permission validation

- Resolve actor permissions before retrieval.
- Re-authorize before tool execution.
- Re-check permissions after tenant switch, role change, or approval state change.

### Data and tenant isolation

- Partition or strictly filter indexes by tenant.
- Include tenant metadata in memory, embeddings, and retrieval results.
- Clear tenant-scoped UI and AI context on tenant switch.
- Never train shared customer-specific behavior across tenants without explicit approved policy.

### Sensitive information protection

- Redact secrets from prompts, context, outputs, telemetry, and audit where necessary.
- Avoid sending restricted data to providers not approved for that data class.
- Apply data minimization to every model request.

### Prompt injection defense

- Treat retrieved content as untrusted.
- Separate system instructions from user and retrieved content.
- Detect attempts to override policy, exfiltrate data, or misuse tools.
- Restrict tools by task and permission.

### Tool execution safety

- Validate tool input schemas.
- Apply risk classification.
- Require explicit approval for medium and high-risk tools.
- Execute through backend domain services.
- Record audit events.

## Part 8 — AI integration

### Projects

AI summarizes project state, risks, ownership, recent changes, and recommended improvements using project-scoped permissions.

### Repositories

AI supports code review assistance, commit and pull request summarization, repository onboarding, secret finding explanation, and documentation discovery. Repository content is treated as untrusted input.

### CI/CD

AI explains failing jobs, summarizes logs, identifies flaky patterns, proposes pipeline improvements, and correlates deployments with incidents.

### Kubernetes

AI interprets workload health, manifests, events, restarts, scheduling failures, policy violations, and namespace or cluster risk.

### Deployments

AI summarizes rollout state, compares versions, evaluates blast radius, suggests rollback criteria, and explains approval requirements.

### Monitoring, logs, and metrics

AI retrieves time-windowed observability signals, summarizes anomalies, explains likely causes, and links to evidence.

### Topology Graph

AI powers topology insights, dependency explanations, blast-radius analysis, root cause hypotheses, and right-side panel recommendations. It must respect graph RBAC and overlay permissions.

### Notifications

AI can summarize incidents and route recommendations to authorized recipients. Notification content is redacted by channel and recipient permissions.

### Audit logs

AI may summarize audit trails for authorized users, support investigations, and explain sequence of actions. Audit access remains permission-gated.

## Part 9 — Future AI roadmap

Future capabilities should build on the same governance, security, and explainability model.

- Agent collaboration for complex investigations.
- Multi-agent workflows with a coordinator and specialist agents.
- Planning agents that produce reviewed operational plans.
- Self-healing systems with strict policy and approval gates.
- Predictive operations for incidents, capacity, and deployment risk.
- Automated root cause analysis across topology, metrics, logs, traces, and deployments.
- Autonomous remediation for low-risk actions with clear rollback.
- Enterprise copilots tailored to security, platform, application, and executive personas.

Future autonomy must be earned through evaluation, monitoring, customer controls, and progressive trust.
