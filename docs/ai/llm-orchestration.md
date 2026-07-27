# SkyOps LLM Orchestration

## Purpose

This document defines how SkyOps orchestrates multiple large language model providers and local models. The orchestration layer provides routing, fallback, cost control, latency optimization, streaming, context window management, model selection, monitoring, and failure recovery.

It works with the AI architecture, prompt framework, memory system, and agent architecture documents.

## Orchestration principles

- SkyOps must be provider-portable.
- Model selection must respect tenant policy, data sensitivity, task risk, quality, latency, and cost.
- The AI Gateway and Model Routing layers hide provider differences from product features.
- No model provider is trusted with data outside customer and platform policy.
- Fallback must not weaken security, privacy, or output quality requirements.
- Model calls must be observable and auditable at metadata level.

## Supported provider classes

### OpenAI

Used for high-quality reasoning, tool-capable workflows, summarization, code review assistance, and interactive copilots where approved by tenant and platform policy.

### Anthropic

Used for reasoning, long-context analysis, safety-sensitive summarization, and agent workflows where approved.

### Google Gemini

Used for multimodal or long-context scenarios where approved and suitable for tenant policy.

### Local LLMs

Used where customers require stronger data locality, offline operation, restricted data processing, or cost-controlled internal tasks. Local models may have lower capability and require stricter evaluation before high-risk use.

### Future providers

Future providers are onboarded through the Model Registry with capability, policy, security, reliability, evaluation, and cost metadata.

## Model registry

The Model Registry is the authoritative catalog of approved models.

Required metadata:

- Provider.
- Model name and version.
- Supported modalities.
- Context window.
- Tool/function support.
- Streaming support.
- JSON or structured output reliability.
- Cost per input and output unit.
- Latency profile.
- Availability and rate limits.
- Region and data residency properties.
- Data retention and training policy.
- Security review status.
- Approved tasks.
- Restricted tasks.
- Evaluation results.
- Deprecation and retirement status.

## Routing strategy

Model routing evaluates:

- Task type.
- Agent type.
- Tenant policy.
- Data classification.
- Required context window.
- Required reasoning depth.
- Required structured output reliability.
- Tool-use requirements.
- Latency budget.
- Cost budget.
- Provider availability.
- Evaluation score for the prompt and task.

Routing examples:

- Small summarization may use a lower-cost low-latency model.
- Incident response may use a stronger reasoning model with streaming.
- Sensitive customer data may route to an approved regional or local model.
- Tool execution planning may require a model with strong structured output performance.

## Fallback strategy

Fallback is controlled and policy-aware.

Fallback rules:

- Never fallback to a provider disallowed by tenant policy.
- Never fallback to a model that cannot handle the data classification.
- Never fallback from a tool-capable structured workflow to an unstructured model without changing the workflow to safe read-only behavior.
- Prefer degraded read-only summaries over unsafe action proposals.
- Preserve audit metadata about fallback reason.

Fallback reasons:

- Provider outage.
- Rate limit.
- Timeout.
- Model retirement.
- Excessive latency.
- Structured output failure.
- Safety filter failure.

## Cost optimization

Cost optimization techniques:

- Route simple tasks to lower-cost models.
- Summarize and compress context before expensive reasoning.
- Use retrieval to avoid sending irrelevant data.
- Cache safe reusable summaries with tenant scope.
- Apply token budgets by tenant, feature, model, and agent.
- Stream only when user experience benefits.
- Stop long-running agent loops early when progress stalls.
- Monitor cost anomalies.

Cost optimization must not remove required safety context, permissions context, or evidence requirements.

## Latency optimization

Latency optimization techniques:

- Use fast models for classification and routing.
- Run independent retrieval operations in parallel at the service layer.
- Stream responses for longer reasoning tasks.
- Precompute embeddings and summaries for stable knowledge.
- Use incremental context loading.
- Cache route decisions and approved prompt metadata.
- Set per-task timeouts and cancellation.

For incident response, first useful output can be streamed while deeper analysis continues.

## Context window strategy

Context windows are limited and must be managed deliberately.

Strategy:

- Prefer relevant context over exhaustive context.
- Rank by tenant scope, permission, freshness, source authority, and task relevance.
- Compress repetitive logs and metrics into summaries.
- Include raw evidence snippets only when necessary.
- Preserve citations and source IDs.
- Use retrieval rounds for follow-up context instead of one oversized prompt.
- Reject or ask for clarification when context cannot fit safely.

## Streaming responses

Streaming improves perceived latency for chat, incident updates, topology insights, and long summaries.

Streaming requirements:

- Stream only safe partial content.
- Do not stream tool execution before validation and approval.
- Support cancellation.
- Preserve final response metadata and citations.
- Handle provider disconnects with clear recovery behavior.
- Avoid exposing internal reasoning or unsafe intermediate tool plans.

## Model selection by workload

### Summarization

Use cost-efficient models with strong factual compression and citation behavior.

### Investigation

Use stronger reasoning models and retrieval-aware workflows.

### Tool planning

Use models with reliable structured output and tool-use performance.

### Code review

Use models evaluated for code understanding, security review, and repository context handling.

### Incident response

Use low-latency streaming plus strong reasoning, with fallback to concise evidence summaries during provider degradation.

### Documentation

Use models optimized for citation-aware writing and consistency with approved documentation.

## Failure recovery

Failure recovery scenarios:

- **Provider timeout:** Retry within budget or fallback to approved model.
- **Rate limit:** Queue, backoff, or route to approved fallback.
- **Invalid structured output:** Ask model to repair once, then fail safely or switch model.
- **Retrieval unavailable:** Continue only if task can be answered without missing evidence; otherwise state limitation.
- **Tool unavailable:** Stop action path and provide manual guidance.
- **Safety policy block:** Refuse unsafe request and explain allowed alternatives.
- **Tenant policy block:** Do not route to disallowed provider; explain configuration requirement when appropriate.

## Observability

LLM orchestration emits telemetry for:

- Provider and model used.
- Prompt and agent version.
- Tenant and feature metadata.
- Latency and streaming duration.
- Token counts and cost.
- Fallback reason.
- Error type.
- Retrieval count and citation coverage.
- Tool proposal and execution metadata.
- Safety and policy denials.

Telemetry must redact sensitive prompt and response contents unless explicitly permitted for debugging under strict access controls.

## Governance

Model governance requires:

- Security review before provider use.
- Legal and privacy review for provider data handling.
- Evaluation before production routing.
- Tenant policy controls.
- Change management for default routes.
- Retirement plan for deprecated models.
- Incident response plan for provider compromise or quality regression.

## Future orchestration roadmap

Future capabilities:

- Per-tenant model preference policies.
- Dynamic routing based on real-time quality and cost telemetry.
- Local model acceleration for sensitive or high-volume tasks.
- Multi-model debate for high-risk recommendations.
- Automatic prompt and model regression detection.
- Model-specific safety adapters.
- Enterprise-managed model endpoints.
