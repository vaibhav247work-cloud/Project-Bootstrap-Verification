# React Application and Module Structure Standard

Use React with TypeScript strict mode and organise by business feature—not by one global `pages`, `services`, or `components` folder.

```text
src/
  app/                 # bootstrap, providers, router, config
  platform/            # api, auth, permissions, errors, telemetry
  shared/              # stable generic ui, hooks, lib, types, constants
  features/
    booking/           # api, pages, components, hooks, state, types, validation, tests
    loyalty/
  assets/ styles/ main.tsx
```

Use `kebab-case` file/folder names and `PascalCase` components. A feature can be smaller when simple; do not create empty folders to satisfy a diagram.

## Import direction

```text
app -> platform / features / shared
feature -> platform / shared / its own files
platform -> shared
shared -> nothing above it
```

A feature may not import another feature's private files. Expose a deliberate small public contract or move genuinely generic code into `shared`. Never use `common`, `helpers`, `utils`, or `misc` as dumping grounds.

## Boundaries

- `pages`: route-level composition/loading/error UI; no raw HTTP or business decision logic.
- `components`: visual/feature UI; no hidden global state or unrelated API calls.
- `api`: DTO/endpoint mapping for the owning feature.
- `state`: client-only feature state, not duplicate server cache.
- `validation`: UI input schemas; server remains trust boundary.
- `platform/auth` and `platform/permissions`: session and UX checks only; backend enforces access.

Routes are centrally registered, substantial route features are lazy-loaded, and URL input is parsed/validated before use. Feature `index.ts` files expose only deliberate public items.

## State location

Use the narrowest owner: component state first; closest shared parent next; feature context/store only when needed; global state only after an ADR. Server state belongs in the query cache and shareable filter/page/tab state belongs in validated URL state when useful.

Components are pure while rendering. Network calls, storage writes, analytics and mutations occur in event handlers, effects that synchronise external systems, or the query layer—not render.

Before a project starts, record router, API contract/client approach, server-state library, any global store, design system/styling, PWA/offline and native-mobile decisions in PRD/ADR.
