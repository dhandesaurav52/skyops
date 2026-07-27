# SkyOps RBAC and Permission Model

## Purpose

This document defines the SkyOps authorization model for roles, permissions, scopes, resource hierarchy, UI gating, service enforcement, and AI-assisted actions. It is a conceptual security design and does not create APIs or schemas.

## Authorization principles

- Backend authorization is the source of truth.
- Roles grant permissions; permissions authorize actions on resources within scopes.
- Tenant, resource ownership, environment sensitivity, policy, and approval state influence decisions.
- Broad roles must not accidentally grant high-risk permissions.
- UI gates are usability aids, not security controls.
- Every privileged decision must be auditable.

## Resource hierarchy

SkyOps authorization evaluates scope using this hierarchy:

1. Organization.
2. Workspace.
3. Project.
4. Environment.
5. Domain resource such as repository, pipeline, deployment, cluster, namespace, registry, webhook, notification channel, topology resource, AI investigation, or API key.

A role granted at a higher scope may apply to child scopes only when explicitly defined. Some sensitive permissions require direct assignment or additional policy even for high-level administrators.

## Scope types

### Organization scope

Controls identity, SSO, SCIM, MFA, billing, organization-wide audit, global integrations, organization roles, and security policy.

### Workspace scope

Controls groups of projects, workspace settings, shared environments, navigation access, and workspace-level reporting.

### Project scope

Controls repositories, pipelines, deployments, artifacts, topology resources, project security findings, and project observability views.

### Environment scope

Controls production, staging, development, and other runtime environments. Production can require stronger approvals, step-up MFA, and stricter AI action policy.

### Resource scope

Controls specific repositories, clusters, namespaces, pipelines, registries, webhooks, and notification channels when fine-grained delegation is required.

## Role catalog

### Organization owner

Highest customer-controlled role. Manages organization settings, owners, identity configuration, billing, security policy, audit visibility, and emergency controls. Requires MFA.

### Organization administrator

Manages most organization operations except owner-only or security-sensitive actions as defined by policy. Requires MFA.

### Security administrator

Manages security policies, audit access, vulnerability settings, incident workflows, token governance, SSO security controls, and compliance evidence. Requires MFA.

### Platform administrator

Manages platform resources such as clusters, environments, integrations, deployment settings, and operational controls.

### Workspace administrator

Manages workspace membership, settings, projects, and workspace-level integrations within delegated scope.

### Project maintainer

Manages project repositories, pipeline configuration, deployment settings, and project resources.

### Developer

Reads project resources, runs development workflows, views relevant logs and pipeline output, and performs non-production actions according to policy.

### SRE/operator

Investigates health, incidents, topology, logs, metrics, traces, deployment state, and performs operational remediation allowed by environment policy.

### Auditor

Reads audit logs, compliance evidence, configuration history, and security posture without mutation permissions.

### Billing administrator

Manages billing, invoices, plan, and payment settings without access to operational secrets or production actions.

### Read-only viewer

Reads non-sensitive resources within assigned scope. Sensitive logs, secrets, security findings, and exports may require additional permissions.

### AI operator

Uses AI investigation and recommendation features. Tool execution requires separate permissions and policy checks.

## Permission taxonomy

Permissions are action-oriented and resource-specific. Conceptual categories include:

- `organization.read`
- `organization.manage_settings`
- `identity.manage_sso`
- `identity.manage_scim`
- `identity.manage_mfa_policy`
- `members.read`
- `members.invite`
- `members.remove`
- `roles.assign`
- `audit.read`
- `audit.export`
- `project.read`
- `project.manage`
- `repository.read`
- `repository.connect`
- `repository.manage`
- `pipeline.read`
- `pipeline.run`
- `pipeline.manage`
- `deployment.read`
- `deployment.approve`
- `deployment.promote`
- `deployment.rollback`
- `environment.read`
- `environment.manage`
- `cluster.read`
- `cluster.connect`
- `cluster.manage`
- `logs.read`
- `metrics.read`
- `traces.read`
- `security_findings.read`
- `security_policy.manage`
- `registry.read`
- `registry.manage`
- `webhook.manage`
- `notification.manage`
- `api_key.create`
- `api_key.revoke`
- `secret_reference.manage`
- `topology.read`
- `topology.manage_views`
- `ai.investigate`
- `ai.propose_action`
- `ai.execute_tool`
- `data.export`

Actual implementation may define finer-grained permissions, but every permission must have an owner, description, risk level, applicable scopes, and audit requirements.

## Permission risk classes

### Low risk

Read non-sensitive metadata, view dashboards, list public project resources within assigned tenant scope.

### Medium risk

Run pipelines, update project settings, manage notifications, view operational logs, manage webhooks, connect repositories.

### High risk

Manage SSO, assign roles, approve production deployments, roll back production, manage clusters, export data, create privileged API keys, manage security policy, execute AI remediation.

### Restricted

Owner transfer, break-glass access, tenant deletion, customer-managed key changes, broad audit export, production secret reference changes, and critical incident controls.

High-risk and restricted permissions require MFA, audit, and often approval or justification.

## Attribute-based constraints

RBAC is combined with attributes:

- Environment criticality.
- Branch protection status.
- Deployment target.
- Resource sensitivity.
- Time-bound access.
- Approval state.
- Incident mode.
- Feature entitlement.
- Data residency.
- Organization security policy.
- AI confidence and action risk.

This hybrid model avoids role explosion while preserving least privilege.

## Authorization decision flow

1. Resolve authenticated actor and actor type.
2. Resolve organization and resource scope.
3. Validate tenant membership.
4. Load role assignments and group-derived grants.
5. Evaluate direct permissions.
6. Apply attribute constraints and security policies.
7. Check feature entitlement.
8. Check approval and MFA requirements.
9. Emit decision telemetry and audit when required.
10. Allow, deny, or require additional approval/step-up.

## Deny behavior

Deny responses must be safe and useful:

- Do not reveal existence of resources the actor cannot know about.
- Explain missing permission when the actor can see the resource but cannot act.
- Include request ID for support.
- Log denied privileged attempts for monitoring.

## UI authorization

Frontend permission summaries should:

- Hide inaccessible navigation.
- Disable visible but unavailable actions with explanatory copy.
- Show approval, policy, and MFA requirements before action.
- Revalidate permissions after tenant switch, role changes, and before mutations.
- Never expose sensitive data merely because a component is hidden.

## Service and worker authorization

Services and workers must authorize:

- The calling service identity.
- The delegated human, service account, or AI actor.
- The tenant and resource scope.
- The requested action.
- The current policy and approval state.

Workers processing events must not assume the producer already authorized all consequences.

## AI authorization

AI actions are governed by RBAC, policy, and approval.

- AI investigation requires permission to read every evidence source used.
- AI summaries must omit unauthorized data.
- AI tool proposals require `ai.propose_action` plus target resource read access.
- AI tool execution requires `ai.execute_tool` plus the underlying resource permission.
- High-impact AI actions require human approval and possibly step-up MFA.
- AI cannot use hidden system privileges to bypass user permissions.

## Topology Graph authorization

Topology authorization applies to graph nodes, edges, overlays, search, live updates, details panel, context menus, shared views, and AI insights.

- Unauthorized nodes are omitted or safely aggregated without revealing identity.
- Edges to unauthorized resources are redacted or represented only as permitted aggregates.
- Metrics, logs, security findings, and deployment overlays require their own permissions.
- Context menu actions are permission-aware and rechecked server-side.

## Delegation and temporary access

Temporary access may be granted for incident response or support workflows.

Requirements:

- Explicit reason.
- Approver.
- Scope.
- Expiration.
- MFA.
- Audit.
- Automatic revocation.
- Post-access review for high-risk scopes.

## Break-glass access

Break-glass is reserved for emergency recovery.

Controls:

- Minimal number of accounts.
- Strong MFA.
- Separate monitoring.
- Mandatory justification.
- Time-limited session.
- Immediate audit and security notification.
- Post-incident review.

## Role governance

Role governance includes:

- Periodic access reviews.
- Detection of unused high-risk permissions.
- Separation of duties for production approval and deployment execution where required.
- Removal of orphaned service accounts and stale API keys.
- Review of group-to-role mappings after identity provider changes.

## Anti-patterns

Avoid:

- Hard-coded role checks instead of permission checks.
- Client-only authorization.
- Broad administrator roles for automation.
- Cross-tenant cache keys.
- Long-lived privileged API keys.
- AI tools that execute using platform superuser privileges.
- Worker jobs that mutate resources without delegated actor context.
