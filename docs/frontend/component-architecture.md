# SkyOps Component Architecture

## Purpose

This document defines how frontend components are organized, composed, documented, and tested in SkyOps. It supports the approved frontend architecture by preventing component sprawl, cross-feature coupling, and inconsistent UI behavior.

## Architectural layers

SkyOps uses a layered component model:

1. **Tokens:** Semantic visual and motion values.
2. **Primitives:** Accessible UI building blocks with no product knowledge.
3. **Layout components:** Page, grid, stack, split pane, scroll area, and shell layout helpers.
4. **Composites:** Reusable product-neutral patterns such as filter bars, empty states, side panels, and metric cards.
5. **Feature components:** Domain-aware UI for a specific product module.
6. **Route views:** Page-level composition and data-bound orchestration.

Dependencies flow downward only. A primitive cannot import a feature component. A composite cannot call a backend API directly. A route view may compose any lower layer that its module owns or is allowed to share.

## Component ownership

Each component must have an owner:

- Design system team owns tokens, primitives, and highly reused composites.
- App shell team owns global navigation, tenant switcher, account menu, notifications entry, and command palette shell.
- Feature teams own domain-specific components and route views.
- Topology team owns graph rendering, graph controls, graph panels, graph stores, and graph-specific overlays.

Ownership determines review responsibility, documentation expectations, and migration accountability.

## Folder placement

Recommended placement:

```text
apps/web/src/shared/design-system/
apps/web/src/shared/components/
apps/web/src/app-shell/
apps/web/src/modules/<feature>/components/
apps/web/src/modules/<feature>/panels/
apps/web/src/modules/<feature>/tables/
apps/web/src/modules/<feature>/forms/
apps/web/src/modules/<feature>/empty-states/
```

Promotion path:

1. Start inside the feature module.
2. Extract to feature subfolder when reused in that module.
3. Promote to shared composite only after cross-module reuse is demonstrated.
4. Promote to primitive only if it is product-agnostic and accessibility behavior is fully specified.

## Props and data boundaries

Components should receive the minimum data needed to render their responsibility.

- Primitives accept presentation variants, accessible labels, and event callbacks.
- Composites accept structured display data and callbacks but avoid backend models when possible.
- Feature components may accept API DTOs only when the DTO is stable and intentionally part of feature presentation.
- Route views adapt API responses into UI models.

This separation reduces churn when backend contracts evolve and makes components easier to test.

## Server and client component strategy

Component placement should reflect rendering needs:

- Server components compose layouts, static content, route metadata, and initial data.
- Client components handle interactions, local state, browser APIs, real-time subscriptions, forms, virtualization, graph rendering, and imperative focus behavior.
- Client boundaries should be as narrow as possible.
- Shared primitives may be client components when accessibility interactions require browser event handling.
- Topology graph canvas/WebGL rendering is client-only, while the surrounding route shell can remain server-rendered.

## State ownership

State belongs at the narrowest useful scope.

- Component state for visual toggles.
- Form state inside form boundaries.
- URL state for shareable filters, selected tabs, time ranges, and topology view mode.
- Query state for server data.
- Feature store for topology viewport, selected nodes, graph layout status, and interaction mode.
- Global providers only for viewer context, tenant context, theme, flags, and notifications.

## Composition patterns

Preferred patterns:

- Compound components for tightly related primitives such as tabs, menu, and dialog.
- Render slots for page headers, side panels, and toolbars.
- Controlled components for filters, tables, and forms that need URL or route coordination.
- Uncontrolled local behavior for simple popovers and menus.
- Explicit loading, empty, error, and disabled props for reusable components.

Avoid deep prop drilling by moving composition closer to the route or by introducing a feature-level context only when multiple sibling components need the same interaction state.

## Error and boundary design

Components should make failure scope explicit.

- Widgets should render local error states.
- Panels should have reset and retry affordances.
- Route views should rely on route-level error boundaries for unrecoverable failures.
- Components that render external content, AI output, logs, manifests, or repository text must sanitize and isolate rendering failures.

## Accessibility contract

Every interactive component must define:

- Keyboard controls.
- Focus entry and return behavior.
- ARIA roles and labels when native semantics are insufficient.
- Screen reader behavior for async updates.
- Disabled versus read-only semantics.
- Reduced-motion behavior.

Graph and canvas-based components need parallel accessible affordances such as searchable lists, details panels, keyboard focus movement, and textual summaries.

## Testing contract

Components require tests appropriate to their layer:

- Primitives: accessibility, keyboard behavior, variants, disabled states, and theming.
- Composites: loading, empty, error, interactions, and responsive behavior.
- Feature components: permission states, data mapping, domain states, and mutation behavior.
- Route views: integration with routing, query parameters, and data boundaries.
- Topology components: graph state reducers, layout worker contracts, virtualization behavior, selection, filtering, and stream updates.

## Storybook contract

Storybook stories should cover realistic states rather than only happy paths.

Required story categories for shared components:

- Default
- Loading
- Empty
- Error
- Disabled
- Long content
- Dark theme
- Keyboard/focus example
- Responsive example when relevant

Topology stories should include small, medium, and large graph datasets with health, metrics, search, filtering, and panel states.

## Documentation contract

Each shared or complex feature component must document:

- Intent and ownership.
- Accepted data shape.
- Supported states.
- Accessibility behavior.
- Performance notes.
- Security considerations when rendering untrusted content.
- Examples of correct and incorrect usage.

## Anti-patterns

Avoid:

- Direct API calls inside design system components.
- Hidden authorization checks inside low-level UI primitives.
- Feature modules importing each other's internal components.
- Global stores for all application data.
- Components that only work in one route but live in shared folders.
- One-off visual variants that bypass tokens.
- Rendering untrusted HTML without sanitization.
- Graph or table components that require loading entire enterprise datasets into memory.
