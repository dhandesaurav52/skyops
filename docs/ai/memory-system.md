# SkyOps AI Memory System

## Purpose

This document defines the SkyOps AI memory architecture. AI memory helps agents maintain useful context across interactions while preserving tenant isolation, security, freshness, user control, and auditability.

Memory is not a substitute for authoritative product data. Authoritative state remains in SkyOps services, databases, event streams, and approved knowledge sources.

## Memory principles

- Memory is scoped by tenant, actor, project, resource, session, and policy.
- Memory must never bypass authorization or tenant isolation.
- Memory must have purpose, owner, retention, and deletion behavior.
- Memory should store durable preferences, summaries, and decisions, not raw sensitive data by default.
- Memory retrieval must be explainable and cite sources where possible.
- Users and organizations need controls for memory enablement and retention.

## Memory types

### Short-term memory

Short-term memory exists during a single AI task or agent run.

Responsibilities:

- Track immediate task state.
- Hold retrieved context summaries.
- Track tool outputs.
- Maintain current hypotheses and next steps.
- Expire at task completion or timeout.

Short-term memory is useful for incident analysis and multi-step investigations but must not persist unless explicitly promoted under policy.

### Conversation memory

Conversation memory preserves context within an ongoing chat or assistant thread.

Responsibilities:

- Remember prior user questions and AI responses.
- Preserve selected tenant, project, resource, and time range.
- Track unresolved follow-ups.
- Support continuity across short interruptions.

Conversation memory expires according to organization policy and user settings.

### Session memory

Session memory is tied to an authenticated user session.

Responsibilities:

- Preserve current workspace, active project, recent resources, graph view, and investigation context.
- Clear on logout, tenant switch, session revocation, or expiration.
- Avoid storing sensitive raw logs, secrets, or credentials.

### Project memory

Project memory stores durable project-specific operational knowledge.

Examples:

- Service ownership notes.
- Runbook associations.
- Known recurring failure patterns.
- Approved deployment conventions.
- Architecture summaries.
- Common investigation paths.

Project memory requires project-level authorization for read and write. It should be reviewable and correctable by authorized users.

### Organization memory

Organization memory stores tenant-wide AI knowledge and preferences.

Examples:

- Preferred cloud providers and regions.
- Security and compliance preferences.
- Naming conventions.
- Standard operating procedures.
- Approved remediation policies.
- AI provider restrictions.

Organization memory is governed by organization administrators and security policy.

### Long-term memory

Long-term memory stores durable knowledge that improves future AI assistance.

Requirements:

- Explicit scope and retention policy.
- Clear provenance.
- Review and deletion support.
- Sensitive data minimization.
- Tenant isolation.
- Reindexing when source content changes.

Long-term memory must not become an uncontrolled archive of customer data.

## Context retrieval

Context retrieval combines explicit request context with relevant memory.

Retrieval steps:

1. Resolve actor and tenant scope.
2. Resolve task intent and resource scope.
3. Identify allowed memory stores.
4. Apply permission and policy filters.
5. Retrieve relevant memory by semantic, keyword, metadata, topology, and time filters.
6. Rank by relevance, freshness, authority, and sensitivity.
7. Return source metadata for explainability.

## Knowledge retrieval

Knowledge retrieval accesses authoritative sources such as documentation, repositories, deployments, Kubernetes state, logs, metrics, traces, security findings, topology relationships, and audit logs.

Memory retrieval and knowledge retrieval must be distinguished in AI responses. Memory can capture prior conclusions or preferences; knowledge sources represent current or historical platform evidence.

## Memory expiration

Every memory entry has retention behavior.

Expiration drivers:

- Time-based retention.
- User deletion.
- Organization policy change.
- Tenant deletion.
- Project archival.
- Source data deletion.
- Permission revocation.
- Security incident response.
- Model or embedding migration.

Expired memory must be removed from retrieval and downstream indexes.

## Memory indexing

Memory may be indexed for retrieval using metadata, keyword, vector, and graph relationships.

Index metadata:

- Tenant ID.
- Workspace and project scope.
- Resource scope.
- Actor or owner.
- Memory type.
- Source and provenance.
- Sensitivity classification.
- Created and updated timestamps.
- Expiration timestamp.
- Embedding model version.
- Permission requirements.

Indexes must support deletion, reindexing, and tenant-scoped filtering.

## Memory security

Security requirements:

- Tenant isolation at storage, index, cache, and retrieval layers.
- Encryption at rest and in transit.
- Permission checks before read and write.
- Redaction of secrets and credentials.
- Audit for memory writes, sensitive reads, exports, and deletions.
- No cross-tenant learning without explicit approved policy.
- Incident response controls for memory purge and retrieval shutdown.

## Memory write policy

AI should not silently store long-term memory for sensitive conclusions.

Write modes:

- **Ephemeral:** Default for task and session context.
- **Suggested:** AI proposes memory for human approval.
- **Automatic low-risk:** Allowed for non-sensitive preferences if organization policy permits.
- **Restricted:** Requires approval for security, production, compliance, or sensitive operational memory.

## Memory quality

Memory must be accurate and maintainable.

Quality controls:

- Source attribution.
- Confidence or verification state.
- Staleness indicators.
- User correction flow.
- Duplicate detection.
- Conflict detection.
- Periodic review for durable organization and project memory.

## Memory and AI agents

Agents may use memory only within their allowed scope.

- Incident Response Agent can use incident history and runbooks.
- Kubernetes Agent can use cluster and namespace operational memory.
- Security Agent can use security policy and findings memory.
- Cost Optimization Agent can use cost preferences and historical recommendations.
- Documentation Agent can use approved documentation and project memory.

Agent memory access must be visible in traces and explainability metadata.

## Memory failure behavior

When memory is unavailable:

- AI should continue with current authorized knowledge when possible.
- Responses should state that durable memory was unavailable if it affects confidence.
- Tool execution must not rely on missing memory for safety checks.
- Monitoring should record memory retrieval failures.
