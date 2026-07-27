# SkyOps Frontend Architecture

## Purpose

This document defines the official frontend architecture for SkyOps. It complements the approved product, software, monorepo, engineering, database, API, and backend architecture documents. It does not redesign those decisions; it translates them into a durable frontend blueprint for every engineer and AI coding agent working on the SkyOps web experience.

SkyOps is an enterprise SaaS platform for platform engineering and DevOps operations. The frontend must make complex infrastructure, delivery, observability, security, and AI workflows understandable, safe, fast, and auditable.

## 1. Frontend philosophy

SkyOps frontend engineering follows these principles:

1. **Operator clarity over visual novelty:** Interfaces must help users understand production systems, risk, ownership, health, and next actions quickly.
2. **Trustworthy automation:** AI recommendations, generated changes, and automated actions must expose evidence, confidence, blast radius, permissions, and rollback paths.
3. **Tenant-safe by design:** Organization, workspace, project, environment, and role context must be visible and enforced throughout the UI.
4. **API-first consumption:** The UI consumes versioned backend contracts through generated or typed clients. It must not encode backend-only business rules as hidden alternatives to service policy.
5. **Progressive disclosure:** Dense DevOps data should start with prioritized summaries, then allow drill-down into logs, metrics, events, policies, manifests, and history.
6. **Resilience under partial failure:** A failed widget must not break an entire workspace. Pages should degrade using scoped errors, skeletons, retries, and stale data indicators.
7. **Performance is product quality:** Dashboards, tables, and topology views must remain responsive for enterprise-scale data.
8. **Accessibility is mandatory:** Keyboard navigation, focus management, semantic structure, contrast, screen reader labels, and reduced-motion support are required.
9. **Documented patterns over ad hoc choices:** Common UI problems must be solved once through documented design system, component, data, and state patterns.

## 2. Next.js application architecture

The primary SkyOps web application should live at `apps/web` in the approved monorepo structure. It should use Next.js with the App Router because SkyOps needs nested layouts, server rendering for authenticated shells, streaming, route-level loading states, and a clean server/client component split.

Recommended application responsibilities:

- `apps/web` owns product UI composition, routing, page-level data orchestration, and browser behavior.
- Shared UI primitives live in a frontend package only after their API stabilizes across multiple product modules.
- Generated API clients come from approved contracts and are imported through a dedicated client layer rather than direct page usage.
- Product modules own their feature screens, feature-specific components, hooks, table definitions, filters, and empty states.
- Cross-cutting providers are installed at the narrowest layout boundary that needs them.

The frontend must treat backend services as authoritative for identity, authorization, tenant scoping, policy decisions, state transitions, audit events, and AI action execution.

## 3. App Router structure

The App Router should model product boundaries and tenant context explicitly.

Recommended route groups:

```text
apps/web/app/
  (marketing)/
  (auth)/
  (app)/
    layout.tsx
    org/[orgSlug]/
      layout.tsx
      workspace/[workspaceSlug]/
        layout.tsx
        dashboard/
        projects/
        topology/
        repositories/
        pipelines/
        deployments/
        environments/
        kubernetes/
        observability/
        security/
        ai/
        settings/
  api/
```

Design decisions:

- Route groups separate public, authentication, and authenticated product surfaces without changing URL semantics.
- Organization and workspace are path segments so deep links carry tenant context.
- Product modules are mounted below workspace routes unless a capability is organization-wide.
- Layouts fetch stable shell data such as viewer, organization, workspace, navigation, feature flags, and permissions.
- Pages fetch view-specific data and delegate complex interactions to client islands.
- Route-level `loading`, `error`, and `not-found` files provide consistent shell behavior.

## 4. Module organization

Frontend modules should align with product domains and backend bounded contexts. A module should contain only UI, state, and presentation orchestration for its domain.

Primary modules:

- `identity` for login-adjacent UI, account settings, sessions, SSO, SCIM surfaces, and API token UX.
- `organizations` for organization settings, workspaces, projects, users, groups, and tenant governance.
- `repositories` for Git provider connections, repository lists, branches, pull requests, and mapping.
- `pipelines` for CI/CD runs, stages, jobs, logs, artifacts, and approvals.
- `deployments` for releases, rollout status, promotion, rollback, and deployment history.
- `environments` for environment inventory, variables, approvals, drift, and protection rules.
- `kubernetes` for clusters, namespaces, workloads, pods, services, ingress, events, and manifests.
- `observability` for metrics, logs, traces, alerts, SLOs, dashboards, and incidents.
- `security` for vulnerabilities, policies, secrets posture, compliance, and risk findings.
- `ai` for copilots, investigations, generated summaries, evidence, and action proposals.
- `topology` for the flagship application topology graph and its supporting panels.

## 5. Feature-based folder structure

Within `apps/web`, use feature-first organization while keeping framework files in `app`.

```text
apps/web/src/
  app-shell/
  modules/
    topology/
      components/
      panels/
      graph/
      data/
      state/
      types/
      empty-states/
      docs/
    pipelines/
    deployments/
  shared/
    components/
    design-system/
    hooks/
    lib/
    providers/
    telemetry/
    testing/
  clients/
  config/
```

Rules:

- Feature code must not import internals from another feature. Cross-feature collaboration goes through shared contracts, page composition, or stable shared components.
- `shared` must contain product-agnostic primitives, not business workflows.
- Feature-specific components stay inside the feature until at least two other modules need them.
- Data access functions are grouped by backend API domain and exposed through typed query helpers.
- Global providers are minimized to avoid unnecessary client-side hydration.

## 6. Shared component strategy

Shared components are grouped by stability:

1. **Primitives:** Button, input, label, dialog, popover, menu, tooltip, tabs, badge, avatar, skeleton, spinner.
2. **Composites:** Page header, command palette, filter bar, data toolbar, confirmation dialog, side panel, metric card, status pill.
3. **Product shells:** App shell, tenant switcher, global navigation, breadcrumbs, permission gate, feature flag gate.
4. **Domain templates:** Deployment status card, pipeline stage timeline, Kubernetes resource identity. These remain shared only if ownership and versioning are clear.

Shared components must be accessible, theme-aware, responsive, documented, and free of hidden API calls. Components that fetch data are feature components, not design system primitives.

## 7. Design system architecture

The design system is defined in `docs/frontend/design-system.md`. Architecturally, it consists of:

- Design tokens for color, typography, spacing, radius, elevation, motion, z-index, and semantic status.
- Primitive components built on accessible foundations.
- Composable patterns for dashboards, lists, forms, side panels, timelines, and graph overlays.
- Content standards for operational states, AI explanations, errors, and destructive actions.
- Governance through Storybook, visual regression tests, accessibility checks, and documented usage guidance.

## 8. UI component hierarchy

SkyOps components follow a strict hierarchy:

1. **Design tokens:** Raw visual decisions represented as named tokens.
2. **Primitives:** Accessible building blocks with no product knowledge.
3. **Layout primitives:** Stack, grid, split pane, scroll area, resizable panel, page frame.
4. **Composites:** Reusable UX patterns assembled from primitives.
5. **Feature components:** Domain-aware components with feature-specific props and behavior.
6. **Route views:** Page-level composition, data loading boundaries, and URL state ownership.

Lower layers must not depend on higher layers. This keeps primitives reusable and prevents product-specific coupling from leaking into the design system.

## 9. State management strategy

Use the smallest state scope that satisfies the problem:

- **URL state:** Search terms, filters, sort order, selected tab, graph view mode, focused resource, time range, and pagination when shareable or restorable.
- **Server state:** API data, cache freshness, mutations, optimistic updates, retries, and invalidation. Use a query library only in client islands that require interactivity; prefer server fetching for stable route data.
- **Local component state:** Open menus, active popovers, form drafts, transient UI toggles.
- **Feature stores:** Complex client-only state such as topology graph viewport, selected nodes, layout mode, and interaction state.
- **Global state:** Viewer, tenant context, theme, feature flags, and notification queue only.

Avoid global stores for API data. Server state should remain cacheable, invalidatable, observable, and tied to backend contracts.

## 10. Server Components vs Client Components

Server Components should be the default for route shells, static composition, authenticated layout data, and read-heavy content. Client Components are used when browser APIs, event handlers, real-time streams, form interactions, canvas/WebGL rendering, drag interactions, or optimistic updates are required.

Guidelines:

- Keep pages and layouts mostly server-rendered.
- Push interactivity into small client islands.
- Do not pass secrets, raw tokens, or privileged backend-only context into client components.
- Use server actions only where they fit approved API and audit patterns; most mutations should go through explicit API clients.
- The topology graph renderer is a client component, but its surrounding shell, permissions, initial query, and metadata can be server-rendered.

## 11. Data fetching strategy

Data fetching must be typed, tenant-scoped, observable, and resilient.

- Server-rendered routes fetch initial data using server-side API clients with authenticated context.
- Client islands fetch incremental or interactive data through typed query helpers.
- Mutations use typed clients, explicit invalidation, optimistic updates only where rollback is safe, and user-visible audit outcomes.
- Long-running operations display progress and link to logs, events, or audit entries.
- Time-series and log data should be windowed, paginated, streamed, or sampled rather than loaded wholesale.
- API errors must preserve request IDs and safe diagnostic metadata.

## 12. Authentication flow

Authentication is handled by the approved identity backend and enterprise identity integrations. The frontend architecture should support:

- Public unauthenticated routes for marketing, documentation links, and login entry.
- Auth routes for SSO initiation, callback handling, account selection, and recovery flows.
- Authenticated app layouts that validate session state before rendering tenant data.
- Session refresh that avoids exposing long-lived credentials to browser code.
- Clear handling for expired sessions, revoked access, MFA requirements, and organization membership changes.

The app shell must never assume that a previously loaded user still has current permissions.

## 13. Authorization and RBAC in UI

Backend authorization is authoritative. UI authorization improves clarity and safety but is not a security boundary by itself.

Patterns:

- Fetch permission summaries with route shell data.
- Hide navigation items the user cannot access.
- Disable actions the user can see but cannot execute, with a reason tooltip.
- Use permission gates for destructive actions, AI actions, production changes, and secret-bearing screens.
- Revalidate permissions before mutations and after tenant switches.
- Display environment protection, approval requirements, and policy blocks near the action.

## 14. API client architecture

The API client layer should centralize transport behavior while keeping domain clients typed and focused.

Responsibilities:

- Base URL and environment resolution.
- Authentication headers through server-safe or browser-safe mechanisms.
- Tenant headers or path context when required by contracts.
- Request IDs and trace propagation.
- Timeout, retry, cancellation, and idempotency-key support.
- Typed request and response models generated from contracts where possible.
- Normalized error envelopes.

Pages and components should not call `fetch` directly except inside the client abstraction itself.

## 15. Error handling strategy

Errors are categorized as validation, authentication, authorization, not found, conflict, rate limit, dependency failure, unavailable, timeout, and unknown. Each category has a standard UI treatment.

- Validation errors appear inline near fields.
- Authentication errors route to login or account recovery.
- Authorization errors explain missing access without revealing sensitive data.
- Conflicts show current state and recovery options.
- Rate limits show retry timing when available.
- Dependency failures preserve page shell and isolate the failed panel.
- Unknown errors show request ID, support path, and safe retry action.

## 16. Loading states

Loading states must communicate scope and preserve layout stability.

- Route transitions use layout-level skeletons.
- Tables use row skeletons with stable headers.
- Dashboards use card skeletons and stale-data badges when refreshing.
- Graph views show staged loading: metadata, topology index, layout, then overlays.
- Buttons show action-specific pending states and prevent duplicate destructive mutations.

## 17. Empty states

Empty states must be specific, actionable, and permission-aware.

Examples:

- No repositories connected: explain Git provider connection and show action if permitted.
- No deployments: link to pipeline setup or environment configuration.
- No topology nodes: explain required telemetry, Kubernetes integration, or filters.
- No search results: show active filters and reset action.
- No permission: explain who can grant access.

## 18. Notifications and toasts

Toasts are for transient confirmations and recoverable notices, not primary error handling. Persistent operational outcomes should appear in page content, audit trails, or activity panels.

Notification levels:

- Success for completed user-initiated actions.
- Info for background activity or AI analysis completion.
- Warning for degraded data, stale caches, or policy-sensitive actions.
- Error for failed actions with retry or details link.

## 19. Forms and validation

Forms should use schema-aligned validation and explicit submission states.

- Client validation improves speed and ergonomics.
- Server validation remains authoritative.
- Field errors map to API error paths.
- Destructive forms require confirmation and show blast radius.
- Production-impacting forms should show policy checks, approvals, and audit implications.
- Draft state should be local unless there is a product requirement for saved drafts.

## 20. Tables and data grids

Tables are core to SkyOps inventory, operations, security, and audit workflows.

Standards:

- Server-side pagination for large datasets.
- URL-backed filters and sort order for shareable views.
- Column definitions owned by features.
- Row identity must be stable.
- Bulk actions must be permission-aware and show selected scope.
- Virtualization is required for large client-side result windows.
- Export must respect tenant, permission, and audit requirements.

## 21. Dashboard architecture

Dashboards are composed of independently loadable cards and panels. Each card owns its loading, empty, error, stale, and refresh states.

Dashboard rules:

- The top summary row shows health, risk, change activity, incidents, and AI-prioritized recommendations.
- Time range is URL-backed and shared by compatible widgets.
- Widgets must state data freshness.
- Cross-widget drill-down uses consistent resource URLs.
- AI summaries must cite source signals and allow opening evidence.

## 22. Navigation and routing

Navigation must reinforce tenant and product context.

- Global navigation includes organization/workspace switcher, search, notifications, help, account, and command palette.
- Primary navigation groups capabilities by operator workflow, not backend service names.
- Breadcrumbs show organization, workspace, project/environment/resource where applicable.
- Deep links are stable and include enough context to restore views.
- Route guards handle missing tenant, disabled feature, and missing permission states.

## 23. Search architecture

SkyOps should provide global and contextual search.

- Global search finds repositories, pipelines, deployments, services, clusters, namespaces, alerts, logs, documentation, and AI investigations.
- Contextual search narrows within a table, graph, log stream, or settings area.
- Search inputs should debounce, cancel stale requests, and expose query syntax when advanced filters exist.
- Search results must show resource type, tenant context, health, recency, and permission-safe snippets.

## 24. Filtering and sorting

Filters should be predictable and shareable.

- Use URL query parameters for meaningful filters.
- Represent filter chips in a standard filter bar.
- Support saved views only when product requirements define ownership and sharing.
- Prefer server-side filtering for large or sensitive datasets.
- Sorting must be stable and clearly indicate unavailable sort modes.

## 25. Theme management

SkyOps supports light and dark themes from the start, with system preference as the default. Theme state is a global user preference but must not block initial rendering.

Requirements:

- Semantic tokens drive all color usage.
- Operational status colors have accessible contrast in both themes.
- Charts, graph edges, and overlays use tokenized palettes.
- Reduced motion and high contrast preferences are respected.

## 26. Responsive design strategy

SkyOps is desktop-first because many workflows involve dense operational data, but it must remain usable on tablets and narrow screens.

- Navigation collapses without losing tenant context.
- Tables transform through column priority, horizontal scroll, or card summaries depending on data type.
- Side panels become modal sheets on small screens.
- Graph view supports inspect-and-drill-down on tablets, while very small screens may show guided list alternatives for complex graph interactions.

## 27. Accessibility standards

SkyOps targets WCAG 2.2 AA.

Mandatory practices:

- Semantic landmarks and headings.
- Keyboard access for all actions.
- Visible focus indicators.
- Screen reader labels for status, charts, graph controls, and icon-only buttons.
- Accessible dialogs, menus, comboboxes, tabs, and tooltips.
- Non-color indicators for health and severity.
- Reduced motion support.
- Automated and manual accessibility testing.

## 28. Internationalization future

The initial product may ship in English, but architecture must not block internationalization.

- Centralize user-facing strings in feature-owned message files when feasible.
- Avoid concatenated strings that cannot be translated.
- Use locale-aware formatting for dates, numbers, durations, and relative time.
- Design layouts for text expansion.
- Keep API enum display names mapped through UI labels.

## 29. Performance optimization

Performance targets:

- Fast authenticated shell render.
- Minimal client JavaScript per route.
- Route-level code splitting by product module.
- Virtualized lists, tables, and graph panels.
- Image and icon optimization.
- Streaming and progressive rendering for heavy pages.
- Web workers for expensive graph layout, diffing, and log parsing.
- Measurements through real user monitoring, synthetic checks, and bundle analysis.

## 30. Caching strategy

Caching must balance freshness with operational correctness.

- Shell metadata can be cached briefly and revalidated on tenant switch or permission change.
- Inventory data may use stale-while-revalidate with visible freshness labels.
- Logs, metrics, and topology health overlays require time-windowed freshness semantics.
- Mutations invalidate affected resources, dashboards, and graph indexes.
- Never cache sensitive data across tenants or users.

## 31. Real-time updates

Use real-time updates where they materially improve operational awareness: pipeline logs, deployment rollout, topology health, alerts, incidents, and AI investigation progress.

Preferred model:

- SSE for one-way event streams from server to browser.
- WebSocket only when bidirectional low-latency interaction is required.
- Events include sequence IDs, resource IDs, timestamps, and tenant context.
- Clients handle reconnect, backoff, missed-event recovery, and visible degraded mode.
- Real-time events update local views through narrow stores rather than global mutation storms.

## 32. File upload architecture

File uploads should use direct-to-object-storage flows when possible.

- The UI requests an upload intent from the backend.
- The backend returns allowed file metadata, size limits, content type rules, and a scoped upload target.
- The UI uploads directly with progress, cancellation, and checksum support.
- The backend finalizes, scans, validates, audits, and attaches uploaded files.
- Failed uploads show recoverable status and do not create partial product state.

## 33. Frontend testing strategy

Testing layers:

- Unit tests for pure formatters, mappers, reducers, and permissions helpers.
- Component tests for primitives, composites, feature widgets, forms, and states.
- Integration tests for route behavior, data fetching, mutations, and authorization rendering.
- End-to-end tests for critical journeys: login, tenant switch, repository connection, pipeline inspection, deployment rollout, topology investigation, and AI action approval.
- Accessibility tests using automated checks plus manual keyboard and screen reader review.
- Visual regression tests for design system and critical dashboard states.

## 34. Storybook strategy

Storybook is the canonical frontend workshop for shared components and complex feature states.

- Every primitive and composite has stories for default, loading, error, empty, disabled, dark theme, and accessibility-relevant states.
- Feature components should include stories when they are reused, complex, or operationally critical.
- Topology graph stories cover scale, health overlays, clustering, search focus, context menus, and degraded streaming.
- Storybook integrates visual regression and accessibility checks in CI.

## 35. Component documentation

Component documentation must answer:

- What problem does this component solve?
- When should it be used or avoided?
- What are accessibility obligations?
- What states are supported?
- What data shape is expected?
- What product or security constraints apply?

Documentation belongs near the component and is surfaced through Storybook where practical.

## 36. Security best practices

Frontend security requirements:

- Do not store long-lived secrets in browser storage.
- Prefer HTTP-only secure cookies for session material where architecture permits.
- Sanitize and safely render logs, markdown, AI output, repository content, and user-provided metadata.
- Enforce CSP-compatible patterns.
- Avoid unsafe HTML injection.
- Treat feature flags as exposure controls, not security boundaries.
- Redact secrets in logs, errors, telemetry, and UI previews.
- Confirm destructive and production-impacting operations.
- Preserve audit context for user actions.

## 37. Error boundaries

Use layered error boundaries:

- Root boundary for unrecoverable app failures.
- Layout boundary for tenant or shell failures.
- Route boundary for page-level failures.
- Panel boundary for dashboards and topology side panels.
- Widget boundary for independent cards.

Boundaries must provide retry, safe diagnostic details, request ID when available, and support links.

## 38. Feature flags

Feature flags enable controlled rollout, experiments, tenant entitlements, and operational kill switches.

Rules:

- Flags are evaluated server-side for route access and shell navigation.
- Client-side flags may adjust visible components but must not grant access.
- Flag names are stable, documented, and owned.
- Removed features require flag cleanup.
- Critical kill switches should degrade UI safely and explain unavailability.

## 39. Frontend observability

Frontend telemetry must help operate SkyOps itself.

Capture:

- Web vitals and route performance.
- API request latency, errors, retries, and request IDs.
- Client exceptions with source maps and redaction.
- User journey events for critical workflows.
- Graph performance metrics such as node count, frame rate, layout time, memory pressure, and event lag.
- Accessibility and usability signals from controlled tests.

Telemetry must respect privacy, tenancy, contractual obligations, and user consent requirements.

## 40. Future extensibility

The architecture supports future growth through:

- Feature modules aligned with product domains.
- Typed client generation from contracts.
- Design system tokens and documented component APIs.
- Route groups for new product surfaces.
- Pluggable dashboard widgets governed by permissions and contracts.
- Topology graph adapters for additional resource types.
- Internationalization-ready content patterns.
- Feature flags and entitlement-aware navigation.

Extensibility must remain deliberate. New abstractions require demonstrated reuse, ownership, and documentation.
