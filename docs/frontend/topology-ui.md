# SkyOps Application Topology Graph UI Architecture

## Purpose

The Application Topology Graph is the flagship SkyOps experience. It gives platform teams, developers, SREs, security teams, and AI agents a shared visual model of applications, infrastructure, delivery activity, dependencies, health, risk, and operational evidence.

The graph must support thousands of nodes while staying responsive, explainable, accessible, and safe for enterprise use.

## Overall graph architecture

The topology UI is a dedicated feature module composed of five layers:

1. **Topology route shell:** Server-rendered tenant, workspace, permissions, feature flags, initial filters, and page metadata.
2. **Topology data layer:** Typed clients and query helpers for graph indexes, resource summaries, health, metrics, logs, deployment history, and AI insights.
3. **Graph state layer:** Client-side store for viewport, selection, hover, expansion, filters, layout mode, loaded clusters, search focus, and streaming status.
4. **Rendering layer:** High-performance canvas or WebGL renderer for nodes and edges, with HTML overlays for controls, panels, menus, and accessible summaries.
5. **Insight layer:** Right-side details panel and AI insights panel that connect selected graph entities to metrics, logs, events, deployments, policy, and recommended actions.

The graph is not a single giant API response. It is an interactive graph workspace that loads summarized topology first, then progressively fetches details, neighborhoods, overlays, and evidence as users navigate.

## Graph rendering approach

The primary renderer should use WebGL or a canvas-backed graph engine for scale. SVG is acceptable only for small embedded diagrams and Storybook examples because thousands of nodes and animated overlays will exceed SVG performance budgets.

Rendering strategy:

- Draw nodes, edges, labels, badges, health rings, and metric overlays in batched canvas/WebGL layers.
- Use HTML only for panels, toolbars, context menus, popovers, command surfaces, and selected-node details.
- Run expensive layout calculations in a Web Worker.
- Use level-of-detail rendering to simplify labels, badges, and edges when zoomed out.
- Use spatial indexing for hit testing, hover, selection, and viewport culling.
- Stream incremental graph diffs rather than re-rendering the whole graph for every update.
- Cap animation and transition work for large graphs to preserve interaction frame rate.

## Data model

The UI should consume a normalized graph model:

- `nodes` keyed by stable resource ID.
- `edges` keyed by stable relationship ID.
- `clusters` keyed by grouping ID.
- `overlays` keyed by resource ID and time range.
- `events` keyed by resource ID and timestamp.
- `metadata` containing tenant context, freshness, version, and layout hints.

The backend remains authoritative for resource identity, relationships, health, metrics, RBAC, and AI evidence. The frontend may compute local layout and visual grouping but must not invent operational truth.

## Node types

Initial node taxonomy:

- Organization, workspace, project, and environment context nodes for high-level grouping.
- Cluster and namespace nodes for Kubernetes scope.
- Workload nodes: deployment, stateful set, daemon set, job, cron job, replica set, pod.
- Network nodes: service, ingress, gateway, endpoint, load balancer, DNS record.
- Delivery nodes: repository, branch, pull request, pipeline, pipeline run, artifact, image, release, deployment.
- Infrastructure nodes: cloud account, region, VPC/network, subnet, database, queue, cache, storage bucket, secret store.
- Observability nodes: alert, incident, SLO, dashboard, trace service, log source.
- Security nodes: vulnerability, policy, finding, secret exposure, compliance control.
- AI nodes: investigation, recommendation, proposed action, evidence bundle.

Node visual encoding:

- Shape communicates resource category.
- Icon communicates resource type.
- Border or ring communicates health.
- Badge communicates alerts, policy blocks, or deployment activity.
- Fill intensity or overlay communicates selected metric when enabled.

## Edge relationships

Edge types must be explicit and filterable:

- Owns or contains.
- Deployed to.
- Runs in.
- Exposes.
- Routes to.
- Depends on.
- Calls.
- Produces.
- Consumes.
- Built from.
- Promoted from.
- Observed by.
- Protected by policy.
- Affected by incident.
- Recommended by AI.

Edges use direction, style, thickness, and badges to communicate meaning. For example, service dependencies can use directional arrows, policy relationships can use dashed lines, and high-traffic edges can vary thickness when metrics overlays are enabled.

## Zoom levels

The graph supports semantic zoom:

1. **Portfolio level:** Workspaces, projects, environments, and high-level health. Individual workloads are hidden.
2. **Environment level:** Clusters, namespaces, major applications, incidents, and release activity.
3. **Application level:** Services, workloads, dependencies, active deployments, and health.
4. **Resource level:** Pods, images, ingress, queues, databases, alerts, and policy findings.
5. **Evidence level:** Logs, traces, metrics, events, manifests, and AI evidence opened in panels rather than cluttering the graph.

At each zoom level, labels and overlays are intentionally reduced or expanded. Users should not be forced to visually parse thousands of labels simultaneously.

## Live updates

Live updates are delivered through SSE by default, with WebSocket reserved for future collaborative or bidirectional graph use cases.

Update types:

- Node added, updated, removed.
- Edge added, updated, removed.
- Health changed.
- Metric overlay changed.
- Deployment event occurred.
- Incident or alert changed.
- AI insight generated or updated.
- Data freshness or stream status changed.

The client applies updates through a graph diff queue. Diffs are batched, ordered by sequence ID, and dropped into a recovery flow when gaps are detected. The UI shows live, reconnecting, stale, or offline stream state.

## Filtering

Filtering must be fast, URL-backed, and explainable.

Core filters:

- Project.
- Environment.
- Cluster.
- Namespace.
- Resource type.
- Owner/team.
- Health.
- Severity.
- Deployment status.
- Policy status.
- Time range.
- Dependency depth.
- Traffic threshold.
- AI confidence.

Filtering should dim or hide unmatched nodes depending on mode. When filters would hide connected context, the graph should offer `show connecting paths` to preserve explainability.

## Search

Search supports resources, labels, annotations, owners, repositories, images, incidents, alerts, and AI evidence.

Behavior:

- Search results appear in a side or command palette list with resource type and status.
- Selecting a result centers and highlights the node.
- Search can expand collapsed clusters to reveal matches.
- Search state is reflected in the URL when meaningful.
- Large graph search uses indexed server results plus client-side visible-node highlighting.

## Cluster view

Cluster view shows Kubernetes clusters as primary containers.

- Clusters display health, region, provider, Kubernetes version, node pressure, and active incidents.
- Namespaces appear as nested groups or expandable clusters.
- Cross-cluster dependencies are emphasized because they often represent latency, risk, or ownership boundaries.
- Cluster-level filters allow users to isolate production, staging, region, provider, or team-owned clusters.

## Namespace view

Namespace view focuses on workloads and services inside a namespace.

- Workloads, services, ingress, config, secrets references, pods, and policies are shown.
- Namespace quotas, policy violations, restarts, and deployment activity appear as overlays.
- The view supports switching between logical service layout and Kubernetes ownership layout.
- Users can open manifests, events, logs, and metrics from selected resources.

## Service dependency view

Service dependency view emphasizes runtime call relationships.

- Services are primary nodes.
- Edges represent observed calls, configured dependencies, queues, or event streams.
- Edge overlays show request rate, latency, error rate, saturation, or traffic direction.
- Users can choose dependency depth from one hop to full reachable graph.
- Critical path and blast-radius highlighting identify affected services during incidents or deployments.

## Deployment timeline

The deployment timeline is integrated below or beside the graph.

Capabilities:

- Show releases, pipeline runs, approvals, rollouts, rollbacks, incidents, and health changes over time.
- Scrub the timeline to see topology state at a point in time.
- Compare before and after deployment health.
- Highlight nodes changed by a selected deployment.
- Link to pipeline logs, release notes, image details, and audit entries.

Timeline data must be windowed and summarized so large deployment histories do not overload the browser.

## AI insights panel

The AI insights panel explains what SkyOps believes is important and why.

Content:

- Incident summaries.
- Suspected root causes.
- Risky deployment correlations.
- Policy and security concerns.
- Blast-radius predictions.
- Remediation suggestions.
- Confidence and uncertainty.
- Evidence links to metrics, logs, traces, events, manifests, and commits.

Rules:

- AI output must be clearly labeled.
- Recommendations must not appear as facts unless supported by evidence.
- Proposed actions require permission checks and explicit user confirmation.
- Production-impacting AI actions must show blast radius, approvals, rollback, and audit implications.

## Right-side details panel

Selecting a node or edge opens a right-side details panel.

Panel sections:

- Identity and ownership.
- Health summary.
- Current status and freshness.
- Related deployments.
- Metrics summary.
- Recent logs and events.
- Alerts and incidents.
- Security and policy findings.
- Dependencies and dependents.
- Manifest or configuration references.
- AI insights and recommended next actions.

The panel loads details progressively. It must support direct links so an investigation can be shared.

## Context menus

Context menus provide resource-specific actions.

Examples:

- Open details.
- Focus neighborhood.
- Show dependencies.
- Show dependents.
- Open logs.
- Open metrics.
- Open traces.
- View manifest.
- View deployment history.
- Compare with previous deployment.
- Create incident.
- Ask AI about this resource.
- Propose remediation.

Menus are permission-aware. Disabled items explain missing permissions, unavailable integrations, or unsupported resource types.

## Health indicators

Health is shown at multiple levels:

- Node ring for current health.
- Cluster aggregate badge for grouped resources.
- Edge health for dependency degradation.
- Timeline markers for health changes.
- Details panel explanation for source signals.

Health must include freshness. A stale healthy status is not equivalent to live healthy status.

## Metrics overlays

Users can overlay metrics onto graph nodes and edges.

Supported overlays:

- CPU, memory, restart count, saturation.
- Request rate, latency, error rate.
- Deployment frequency and rollback rate.
- Vulnerability count and severity.
- Policy violations.
- Incident impact.
- Cost or capacity signals in future phases.

Overlays must include units, time range, color legend, and data freshness. For large graphs, overlays should aggregate at cluster or namespace level until users zoom in.

## Log integration

The graph connects directly to logs without turning the graph into a log viewer.

- Details panel shows recent representative log events.
- `Open logs` opens a filtered log view using tenant, resource, namespace, pod, service, time range, and correlation IDs.
- Live log snippets may stream for selected resources only.
- Logs are sanitized and rendered as untrusted text.
- AI insights can cite log evidence with timestamps and links.

## User interactions

Required interactions:

- Pan and zoom.
- Fit to screen.
- Center on selected resource.
- Expand and collapse clusters.
- Select node or edge.
- Multi-select where bulk comparison is useful.
- Hover preview with minimal metadata.
- Keyboard navigation through visible graph entities.
- Command palette actions.
- Context menu actions.
- Drag-free investigation by clicking dependencies and breadcrumbs.
- Save or share view through URL state.

The graph must provide accessible alternatives for users who cannot operate spatial canvas interactions, including searchable lists, keyboard focus order, and details panel summaries.

## Performance strategy for thousands of nodes

Performance requirements:

- Progressive graph loading by scope and zoom level.
- Server-side topology indexing and summarization.
- Viewport culling and level-of-detail rendering.
- Batched WebGL/canvas drawing.
- Web Worker layout computation.
- Incremental layout updates instead of full recomputation.
- Edge bundling or aggregation at high zoom-out levels.
- Cluster collapsing by default for large datasets.
- Debounced filters, search, and resize handling.
- Memory caps for historical overlays and event buffers.
- Telemetry for frame rate, layout time, render time, node count, edge count, and stream lag.

Large enterprise topologies should initially render grouped summaries, not every pod and edge. Users progressively expand into details.

## Security and RBAC

The topology graph must respect permissions at every layer.

- Users only receive graph resources they are permitted to see.
- Sensitive metadata such as secrets, private repository details, and restricted namespaces is redacted or omitted.
- Actions in context menus are permission-checked before rendering and again before execution.
- AI prompts and evidence must not leak resources outside the user's tenant or permissions.
- Shared graph URLs must not bypass authorization.

## Failure modes

The graph must handle partial failure gracefully:

- Topology index unavailable: show retry and link to related inventory pages.
- Metrics unavailable: keep graph visible and mark metrics overlay degraded.
- Logs unavailable: hide live snippets and show integration status.
- Stream disconnected: continue with stale data and show reconnecting state.
- Layout worker failure: fall back to simpler grouped layout.
- Too many nodes: prompt user to filter, cluster, or zoom instead of freezing the browser.

## Extensibility

The topology architecture supports future resource types by adding typed node render definitions, edge definitions, details panel sections, and overlay providers. New graph concepts must define identity, permissions, visual encoding, filters, search behavior, details content, and performance impact before entering the main graph.
