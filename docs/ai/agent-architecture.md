# SkyOps AI Agent Architecture

## Purpose

This document defines the specialized AI agent architecture for SkyOps. Agents are governed AI capabilities that investigate, explain, recommend, and prepare actions across platform engineering and DevOps workflows.

Agents do not own authoritative platform state. They operate through the AI Gateway, Context Management, Memory Management, Model Routing, and Tool Execution Layer described in `docs/ai/ai-architecture.md`.

## Agent design principles

- Agents are specialized by domain and task type.
- Agents operate with least-privilege tools and context.
- Agents must produce explainable outputs with evidence.
- Agents must stop when permissions, policy, confidence, or context are insufficient.
- Agents cannot bypass RBAC, tenant isolation, environment protections, or approval workflows.
- Agents should be composable for future multi-agent workflows.

## Standard agent contract

Every agent definition includes:

- Purpose.
- Responsibilities.
- Capabilities.
- Inputs.
- Outputs.
- Required tools.
- Required permissions.
- Memory access scope.
- Risk classification.
- Limitations.
- Evaluation requirements.
- Future expansion path.

## Agent runtime responsibilities

The Agent Runtime:

- Selects an agent based on task intent.
- Provides authorized context and memory.
- Limits available tools to the task and permissions.
- Tracks task steps and tool proposals.
- Streams safe partial results where useful.
- Requires approval for governed actions.
- Emits telemetry and audit events.
- Handles cancellation, timeout, and failure recovery.

## Specialized agents

### Platform Engineering Agent

**Purpose:** Help platform teams understand and improve the internal developer platform, environments, golden paths, service ownership, and operational workflows.

**Responsibilities:** Summarize platform health, identify workflow bottlenecks, explain service ownership, recommend platform improvements, and connect projects to approved runbooks.

**Capabilities:** Project analysis, environment comparison, runbook discovery, developer-experience summaries, policy explanation, and topology-aware recommendations.

**Inputs:** Project metadata, environment configuration, topology summaries, deployment history, platform documentation, user questions, and organization memory.

**Outputs:** Evidence-backed summaries, recommended improvements, workflow explanations, and action proposals requiring review when changes are needed.

**Limitations:** Cannot modify platform policy, roles, environments, or production configuration without explicit permissions and approval.

**Future expansion:** Golden path generation, service maturity scoring, and platform roadmap recommendations.

### DevOps Agent

**Purpose:** Assist with operational delivery workflows across repositories, pipelines, artifacts, deployments, and environments.

**Responsibilities:** Explain delivery status, diagnose failed releases, compare deployment versions, summarize changes, and recommend rollback or retry options.

**Capabilities:** Pipeline log summarization, deployment timeline analysis, artifact metadata review, environment readiness checks, and release risk explanation.

**Inputs:** Pipeline runs, deployment records, artifacts, commit metadata, approval state, environment policy, logs, and metrics.

**Outputs:** Failure explanations, release summaries, risk assessments, and proposed next steps.

**Limitations:** Cannot approve or execute production changes without required permissions, policy checks, and approvals.

**Future expansion:** Predictive deployment risk and automated low-risk release checks.

### Kubernetes Agent

**Purpose:** Help users understand Kubernetes resource health and operational issues.

**Responsibilities:** Analyze workloads, pods, services, ingress, events, namespaces, resource pressure, policy violations, and deployment health.

**Capabilities:** CrashLoopBackOff investigation, scheduling failure analysis, manifest explanation, namespace health summaries, and topology-aware dependency context.

**Inputs:** Cluster, namespace, workload, pod, service, ingress, events, metrics, logs, manifests, and topology graph context.

**Outputs:** Root-cause hypotheses, evidence links, recommended checks, and safe remediation proposals.

**Limitations:** Cannot execute cluster mutations or access restricted namespaces unless authorized and approved.

**Future expansion:** Automated safe remediation for low-risk restart, scaling, or rollout checks under strict policy.

### Docker Agent

**Purpose:** Assist with container image, Dockerfile, registry, SBOM, provenance, and runtime image concerns.

**Responsibilities:** Explain image build failures, identify insecure image patterns, summarize SBOM and vulnerability findings, and recommend safer base images or build practices.

**Capabilities:** Image metadata analysis, vulnerability explanation, layer and tag risk explanation, registry policy review, and artifact provenance interpretation.

**Inputs:** Dockerfiles, image metadata, registry data, SBOMs, vulnerability findings, pipeline logs, and deployment references.

**Outputs:** Risk summaries, remediation guidance, and image promotion recommendations.

**Limitations:** Cannot push, delete, or promote images without registry and deployment permissions.

**Future expansion:** Automated image hardening recommendations and policy-aware base image selection.

### Terraform Agent

**Purpose:** Assist with infrastructure-as-code review, drift explanation, plan summaries, and change risk.

**Responsibilities:** Summarize Terraform/OpenTofu plans, detect risky infrastructure changes, explain drift, and link infrastructure changes to deployments and incidents.

**Capabilities:** Plan summarization, policy result explanation, drift triage, blast-radius analysis, and change review assistance.

**Inputs:** IaC files, plan outputs, policy checks, cloud inventory summaries, deployment context, and audit history.

**Outputs:** Change summaries, risk levels, affected resources, and approval recommendations.

**Limitations:** Cannot apply infrastructure changes without explicit approvals and environment protections.

**Future expansion:** Predictive infrastructure risk and automated policy remediation suggestions.

### AWS Agent

**Purpose:** Assist with AWS-specific operational, security, and cost questions where AWS integrations are enabled.

**Responsibilities:** Explain AWS resources, identify configuration risks, summarize incidents affecting AWS resources, and connect cloud resources to projects and topology.

**Capabilities:** Cloud inventory summarization, IAM risk explanation, network path context, cost and capacity observations, and region/service health correlation.

**Inputs:** AWS integration metadata, cloud inventory, metrics, security findings, topology relationships, and organization policy.

**Outputs:** Cloud risk summaries, dependency explanations, and recommended investigation steps.

**Limitations:** Cannot make AWS changes directly outside approved SkyOps tools and permissions.

**Future expansion:** Multi-cloud specialist agents and cloud-specific remediation playbooks.

### CI/CD Agent

**Purpose:** Specialize in pipeline execution, logs, flakes, approvals, and release automation.

**Responsibilities:** Diagnose failed jobs, summarize pipeline runs, detect recurring failure patterns, explain approvals, and recommend pipeline improvements.

**Capabilities:** Log clustering, failure pattern detection, retry recommendation, pipeline configuration review, and stage-level analysis.

**Inputs:** Pipeline definitions, job logs, artifacts, previous runs, repository changes, secrets policy, and runner metadata.

**Outputs:** Failure causes, flake likelihood, recommended retries or fixes, and evidence links.

**Limitations:** Cannot expose masked secrets or run privileged jobs without permissions.

**Future expansion:** Autonomous flaky test quarantine recommendations and pipeline optimization planning.

### Git Agent

**Purpose:** Assist with repository, branch, pull request, commit, and code review workflows.

**Responsibilities:** Summarize changes, explain repository activity, identify risky changes, assist code review, and link commits to deployments and incidents.

**Capabilities:** Pull request summaries, commit impact analysis, branch protection explanation, ownership lookup, and change-risk scoring.

**Inputs:** Repository metadata, pull requests, commits, diffs where authorized, code ownership, pipeline status, and deployment history.

**Outputs:** Review summaries, risk notes, questions for reviewers, and deployment impact context.

**Limitations:** Repository content is untrusted; generated review comments require human judgment.

**Future expansion:** Policy-aware review automation and change impact prediction.

### Observability Agent

**Purpose:** Analyze logs, metrics, traces, alerts, SLOs, dashboards, and incidents.

**Responsibilities:** Detect anomalies, summarize incidents, correlate signals, explain service health, and recommend investigation paths.

**Capabilities:** Time-windowed signal analysis, log summarization, trace path explanation, alert clustering, SLO burn interpretation, and topology correlation.

**Inputs:** Metrics, logs, traces, alerts, incidents, topology relationships, deployment timeline, and service ownership.

**Outputs:** Hypotheses, evidence links, affected resources, and next investigative steps.

**Limitations:** Cannot claim root cause without sufficient evidence and should report uncertainty.

**Future expansion:** Predictive incident detection and automated root cause analysis.

### Security Agent

**Purpose:** Help security teams understand findings, policy results, vulnerabilities, access changes, and suspicious activity.

**Responsibilities:** Summarize security posture, explain vulnerabilities, investigate audit events, correlate findings with deployments and topology, and recommend remediation.

**Capabilities:** Vulnerability explanation, policy failure analysis, suspicious activity summaries, access review assistance, and incident support.

**Inputs:** Security findings, SBOMs, audit logs, identity changes, policies, repository metadata, deployment history, and runtime signals.

**Outputs:** Risk summaries, evidence, remediation options, and approval-aware action proposals.

**Limitations:** Cannot change roles, policies, or secrets without privileged permissions and step-up controls.

**Future expansion:** Continuous security posture copilot and automated exception review assistance.

### Cost Optimization Agent

**Purpose:** Help identify waste, capacity imbalance, and cost-risk tradeoffs.

**Responsibilities:** Summarize cost drivers, recommend rightsizing, identify idle resources, and connect cost to project and environment ownership.

**Capabilities:** Cost trend explanation, utilization comparison, cluster and service cost attribution, and savings recommendation prioritization.

**Inputs:** Cost data, resource inventory, metrics, topology ownership, environment tags, and organization policy.

**Outputs:** Savings opportunities, risk tradeoffs, owner-specific recommendations, and expected impact.

**Limitations:** Cannot modify capacity or infrastructure without approved workflows.

**Future expansion:** Predictive budget alerts and automated low-risk savings recommendations.

### Incident Response Agent

**Purpose:** Assist responders during active incidents.

**Responsibilities:** Summarize incident timeline, correlate changes and alerts, identify affected services, recommend containment, and prepare status updates.

**Capabilities:** Timeline reconstruction, blast-radius analysis, deployment correlation, topology exploration, runbook matching, and communication drafting.

**Inputs:** Incident data, alerts, topology graph, logs, metrics, traces, deployments, audit events, and runbooks.

**Outputs:** Incident briefings, hypotheses, evidence links, recommended actions, and stakeholder update drafts.

**Limitations:** Cannot execute containment or remediation without approvals and clear authorization.

**Future expansion:** Multi-agent incident war room and supervised self-healing workflows.

### Documentation Agent

**Purpose:** Help users find, summarize, and draft documentation.

**Responsibilities:** Answer questions using approved docs, generate draft runbooks, summarize architecture, and identify documentation gaps.

**Capabilities:** Documentation retrieval, runbook drafting, architecture summarization, onboarding assistance, and change summary generation.

**Inputs:** Approved documentation, project memory, repository metadata, runbooks, and user context.

**Outputs:** Documentation answers, citations, draft content, and gap reports.

**Limitations:** Drafts are not automatically approved documentation.

**Future expansion:** Documentation freshness checks and auto-generated onboarding guides.

### Code Review Agent

**Purpose:** Assist reviewers with code, configuration, IaC, pipeline, and security review.

**Responsibilities:** Summarize changes, flag risks, identify missing tests, explain security concerns, and connect code changes to operational impact.

**Capabilities:** Diff summarization, pattern detection, policy guidance, test gap identification, and deployment impact analysis.

**Inputs:** Pull request metadata, diffs where authorized, test results, pipeline status, ownership, security findings, and documentation.

**Outputs:** Review notes, questions, risk summaries, and suggested follow-up tasks.

**Limitations:** Does not replace required human review and must not approve changes autonomously.

**Future expansion:** Organization-specific review policy learning and cross-repository impact analysis.

## Multi-agent collaboration

Future multi-agent workflows should use a coordinator agent that delegates narrowly scoped tasks to specialists.

Rules:

- Coordinator cannot grant additional permissions to specialists.
- Specialists return evidence and confidence, not hidden conclusions.
- Shared memory is scoped to the workflow and tenant.
- Final recommendations identify which agents contributed.
- High-risk action proposals require human approval.

## Agent evaluation

Agents are evaluated on:

- Task success.
- Groundedness.
- Evidence quality.
- Safety and permission compliance.
- Tool-use correctness.
- Latency and cost.
- User satisfaction.
- Robustness against prompt injection.
- Domain-specific correctness.

## Agent failure behavior

Agents must fail safely.

- If context is missing, ask for clarification or state uncertainty.
- If permission is denied, explain permitted alternatives.
- If a tool fails, surface the error safely and provide retry guidance.
- If confidence is low, avoid decisive operational claims.
- If policy blocks an action, stop and explain the policy requirement.
