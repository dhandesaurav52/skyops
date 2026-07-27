# SkyOps Prompt Framework

## Purpose

This document defines the standardized prompt framework for SkyOps AI features, services, and agents. Prompts are product-critical artifacts and must be governed with the same discipline as code, policies, and contracts.

The prompt framework works with the AI architecture, memory system, agent architecture, and LLM orchestration documents.

## Prompt principles

- Prompts must be versioned, owned, reviewed, tested, and observable.
- Prompts must preserve tenant isolation and authorization boundaries.
- Prompts must separate platform instructions, task instructions, user input, retrieved context, and tool results.
- Prompts must require evidence and uncertainty handling for operational claims.
- Prompts must not encourage bypassing RBAC, approvals, policy, or security controls.
- Prompts must treat repository content, logs, manifests, tickets, documentation, and AI memory as untrusted input.

## Prompt lifecycle

1. **Proposal:** A feature team proposes a prompt with purpose, owner, agent or feature, expected inputs, outputs, risk level, and evaluation plan.
2. **Design review:** AI, security, product, and domain owners review task framing, safety constraints, data access, and output requirements.
3. **Authoring:** Prompt is written using approved template structure and variable definitions.
4. **Validation:** Prompt variables, required context, tool permissions, and output schemas are validated.
5. **Evaluation:** Prompt is tested against curated scenarios, adversarial examples, regression cases, and quality scorecards.
6. **Approval:** Prompt version is approved for a rollout stage.
7. **Deployment:** Prompt is released behind feature flags and routing policy.
8. **Monitoring:** Quality, cost, safety, latency, user feedback, and incident signals are tracked.
9. **Iteration:** Changes create new versions with changelog and evaluation evidence.
10. **Retirement:** Obsolete prompt versions are disabled and archived with migration notes.

## Prompt template structure

A standard prompt template contains conceptual sections:

- **System role:** Stable platform role, safety boundaries, and non-negotiable policy constraints.
- **Task definition:** What the AI must accomplish for this feature or agent.
- **Tenant and scope context:** Organization, workspace, project, environment, resource, and time range metadata allowed for this request.
- **User request:** The user-provided question or instruction.
- **Retrieved context:** Authorized knowledge, memory, events, logs, metrics, topology, documentation, or tool results.
- **Tool policy:** Available tools, required permissions, risk level, and approval behavior.
- **Output contract:** Required structure, citations, confidence, warnings, next actions, and failure behavior.
- **Safety reminders:** Prompt injection resistance, uncertainty handling, and sensitive data treatment.

Templates must not include implementation code. They describe behavior, constraints, and output expectations.

## Prompt versioning

Prompt versions are immutable once approved for production.

Version metadata:

- Prompt ID.
- Version.
- Owner.
- Feature or agent.
- Risk classification.
- Supported models.
- Required context variables.
- Available tools.
- Evaluation dataset version.
- Approval status.
- Rollout flags.
- Changelog.
- Retirement status.

A prompt change that affects safety, tool execution, authorization, output structure, or data handling requires a new version and review.

## Prompt testing

Prompt testing covers:

- Happy-path task completion.
- Missing context.
- Conflicting context.
- Stale data.
- Permission-denied context.
- Prompt injection attempts.
- Secret leakage attempts.
- Incorrect tool arguments.
- Hallucination traps.
- Ambiguous user requests.
- Provider fallback behavior.
- Output schema compliance.

Testing must include domain-specific examples for repositories, CI/CD, Kubernetes, deployments, observability, security findings, topology graph insights, and incident response.

## Prompt validation

Prompt validation verifies:

- Required variables are defined.
- Variables have safe types and size limits.
- Retrieved context includes source metadata.
- Tool names and permissions are valid.
- Output contract is explicit.
- Safety instructions are present for risky tasks.
- No hard-coded tenant, user, secret, or environment-specific data is embedded.
- Prompt is compatible with approved model routes.

## Prompt storage

Prompt storage must support governance and traceability.

Requirements:

- Prompts are stored in an approved repository or prompt registry.
- Production prompt versions are immutable.
- Access to edit prompts is restricted.
- Changes are reviewed and auditable.
- Prompt deployments are tied to feature flags or release records.
- Sensitive test cases are protected according to data classification.

## Prompt governance

Governance roles:

- **Prompt owner:** Accountable for prompt behavior and lifecycle.
- **Domain reviewer:** Validates task accuracy and terminology.
- **Security reviewer:** Validates tenant isolation, prompt injection defense, sensitive data handling, and tool safety.
- **AI platform reviewer:** Validates model fit, evaluation quality, routing, and observability.
- **Product reviewer:** Validates user experience and expected behavior.

High-risk prompts require stronger review, especially prompts that can trigger tools, recommend production actions, summarize sensitive findings, or influence security decisions.

## Prompt security

Security requirements:

- Treat user input and retrieved content as untrusted.
- Never allow retrieved content to override system or policy instructions.
- Do not include secrets in prompts unless explicitly permitted by policy and redacted where possible.
- Minimize data sent to model providers.
- Preserve tenant and permission scope in context.
- Require tool execution through the Tool Execution Layer.
- Include refusal behavior for unauthorized or unsafe requests.
- Log prompt metadata and safety events without exposing sensitive prompt contents unnecessarily.

## Prompt optimization

Prompt optimization must preserve safety and explainability.

Optimization goals:

- Reduce unnecessary tokens.
- Improve retrieval focus.
- Improve output consistency.
- Lower latency and cost.
- Improve citation coverage.
- Reduce hallucinations.
- Improve tool argument quality.

Optimization cannot remove safety constraints, authorization reminders, or evidence requirements without review.

## Prompt evaluation

Evaluation combines automated and human review.

Evaluation dimensions:

- Correctness.
- Groundedness.
- Citation quality.
- Completeness.
- Safety.
- Permission compliance.
- Tool-use correctness.
- Clarity.
- Latency.
- Cost.
- Robustness against prompt injection.

Evaluation outputs are stored as scorecards and referenced during release review.

## Prompt quality standards

A production prompt must:

- Have an owner and approved version.
- Define expected inputs and output shape.
- Include safety and tenant isolation constraints.
- Require evidence when answering operational questions.
- Handle uncertainty and missing data.
- Avoid hidden implementation assumptions.
- Pass prompt injection and permission-denial tests.
- Be monitored after rollout.

## Prompt categories

### Summarization prompts

Used for incidents, deployments, logs, audit trails, pull requests, security findings, and topology panels. They must cite evidence and avoid adding unsupported conclusions.

### Investigation prompts

Used by agents to analyze failures, incidents, and anomalies. They must preserve assumptions, rank hypotheses, and identify missing evidence.

### Recommendation prompts

Used to propose next actions. They must describe risk, blast radius, confidence, alternatives, and required approvals.

### Tool-planning prompts

Used to prepare tool calls. They must produce schema-valid proposals and stop before execution when approval is required.

### Documentation prompts

Used to help explain platform behavior and generate drafts. They must distinguish generated drafts from approved documentation.

## Rollback and incident handling

Prompt rollback is required when monitoring shows safety regressions, hallucinations, cost spikes, permission leaks, tool misuse, or user harm.

Rollback controls:

- Disable prompt version through feature flag or routing policy.
- Revert to previous approved version.
- Preserve audit evidence.
- Notify owners.
- Run incident review for severe failures.
