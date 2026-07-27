# SkyOps Design System

## Purpose

The SkyOps design system defines the visual language, interaction patterns, accessibility rules, and documentation standards for the platform. It exists to make the product coherent across infrastructure, CI/CD, observability, security, and AI workflows without slowing feature teams.

## Design principles

- **Operational confidence:** Visual design must reveal state, risk, ownership, recency, and available actions.
- **Consistency before customization:** Teams should compose approved primitives and patterns before creating new UI behavior.
- **Accessible by default:** Components must satisfy keyboard, focus, contrast, semantic, and assistive technology requirements by construction.
- **Density with hierarchy:** SkyOps presents dense data, but spacing, typography, grouping, and progressive disclosure must preserve scanability.
- **AI transparency:** AI-generated UI states must distinguish recommendation, evidence, uncertainty, and action.

## Token architecture

Design tokens are the source of truth for visual decisions. Tokens should be semantic rather than tied to one raw color or pixel value.

Token groups:

- **Color:** background, foreground, surface, border, muted, accent, destructive, warning, success, info, critical, neutral, selected, focus, graph-node, graph-edge, metric-overlay.
- **Typography:** font families, sizes, line heights, weights, letter spacing, code text, numeric tabular text.
- **Spacing:** layout scale, component padding, grid gaps, panel gutters, toolbar density.
- **Radius:** none, small, medium, large, pill, card, modal.
- **Elevation:** surface layering, popover shadow, modal backdrop, graph panel depth.
- **Motion:** duration, easing, reduced-motion alternatives, graph transition timing.
- **Z-index:** shell, sticky header, dropdown, popover, modal, toast, command palette, graph overlay.
- **Status:** health, severity, policy outcome, deployment state, pipeline state, incident state, AI confidence.

All product UI must consume semantic tokens. Hard-coded colors, one-off spacing, and custom shadows are not acceptable outside prototypes.

## Theme model

SkyOps supports light, dark, system, and future high-contrast modes.

Requirements:

- Theme selection is persisted per user and applied early to avoid visual flash.
- Status tokens must remain distinguishable without relying only on color.
- Charts and topology overlays use theme-aware palettes.
- AI-generated content, code blocks, logs, and terminal-like output have dedicated readable tokens.
- Reduced-motion preferences disable nonessential animation and graph transitions.

## Typography

Typography must support fast scanning of operational information.

- Page titles identify scope and resource.
- Section headings structure complex dashboards.
- Body text is concise and action-oriented.
- Monospace text is used for resource names, commit SHAs, image digests, log lines, manifests, and code-like values.
- Numeric metrics use tabular alignment where comparison matters.

## Layout system

The layout system should provide consistent building blocks:

- App shell with global navigation, tenant switcher, command palette, notifications, account menu, and help entry.
- Page frame with header, actions, breadcrumbs, tabs, and content area.
- Split pane for master-detail and topology-detail workflows.
- Resizable panels for logs, graph details, and AI insights.
- Card grid for dashboards.
- Sticky toolbars for tables and graph controls.

## Component categories

### Primitives

Primitives are product-agnostic accessible components: button, link, input, select, checkbox, radio, switch, textarea, label, badge, avatar, icon, tooltip, popover, dialog, menu, tabs, accordion, skeleton, spinner, progress, toast, and alert.

Primitives must not fetch data, know tenant context, or encode product permissions.

### Composites

Composites solve recurring UI patterns: page header, empty state, error state, confirmation dialog, side panel, filter bar, search box, data toolbar, status pill, metric card, timeline, activity feed, code block, log viewer shell, and command palette item.

Composites may accept domain labels and callbacks but should not contain backend-specific behavior.

### Product patterns

Product patterns are domain-aware and can depend on permissions, feature flags, API models, and telemetry. Examples include deployment approval panels, pipeline run summaries, Kubernetes resource badges, vulnerability severity summaries, AI evidence cards, and topology graph overlays.

## Status and severity language

SkyOps must use consistent operational vocabulary.

Health states:

- Healthy
- Degraded
- Unhealthy
- Unknown
- Syncing
- Stale

Severity states:

- Critical
- High
- Medium
- Low
- Informational

Policy states:

- Allowed
- Requires approval
- Blocked
- Warning
- Not evaluated

Each state has visual tokens, icon guidance, assistive labels, and content guidance.

## Iconography

Icons should clarify resource type, action, or state, not decorate. Icons must have accessible labels when they are the only visible indicator. Product icons should distinguish repositories, pipelines, deployments, environments, clusters, namespaces, workloads, services, incidents, policies, vulnerabilities, logs, metrics, traces, and AI insights.

## Motion

Motion is used sparingly for orientation and feedback:

- Route transitions should be subtle.
- Loading indicators should not distract from active investigation.
- Topology graph transitions may animate layout changes when node counts are small, but large graphs should prioritize frame rate.
- Reduced-motion users receive instant or cross-fade alternatives.

## Content standards

Content should be direct, specific, and operationally useful.

- Buttons use verbs and objects: `Approve deployment`, `Rollback release`, `Open logs`.
- Empty states explain why the state exists and what to do next.
- Errors include recovery actions and request ID when available.
- AI insights identify source signals, confidence, and whether an action is proposed or already executed.
- Destructive actions describe blast radius and reversibility.

## Accessibility requirements

Every design system component must support:

- Keyboard operation.
- Visible focus styles.
- Semantic roles and labels.
- Screen reader announcements for state changes where appropriate.
- WCAG 2.2 AA contrast.
- Non-color distinction for critical statuses.
- Logical tab order.
- Escape and outside-click behavior for overlays where appropriate.

## Documentation requirements

Each shared component should document:

- Purpose.
- Anatomy.
- Props and variants at a conceptual level.
- Accessibility behavior.
- Keyboard interactions.
- Loading, empty, error, disabled, and read-only states.
- Do and do-not examples.
- Related components.

## Governance

Design system governance ensures consistency without becoming a bottleneck.

- New primitives require design and engineering review.
- Product-specific components must prove reuse before promotion to shared composites.
- Deprecated components require migration guidance.
- Storybook is the source of interactive documentation.
- Visual regression protects token and component changes.
- Accessibility checks are required in CI for shared components.
