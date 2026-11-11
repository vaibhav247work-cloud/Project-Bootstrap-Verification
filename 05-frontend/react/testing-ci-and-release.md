# React Testing, CI and Release Standard

## Default toolchain

- **Vitest** for unit and hook/utility tests, or an approved equivalent.
- **React Testing Library** for component and interaction tests based on accessible user-visible behaviour.
- **Playwright** for critical end-to-end workflows, browser/device projects, visual comparison where useful, and trace/artifact capture.
- An accessibility checker in component/E2E checks, plus manual keyboard and screen-reader testing. Automation does not prove WCAG conformance.

An alternate tool requires an ADR showing equivalent capability. Tools do not substitute for well-chosen test scenarios.

## Test layers

| Layer | Purpose | Examples |
| --- | --- | --- |
| Unit | Pure logic/edge cases | formatters, permission helper, query key, validation schema, reducer |
| Component | User-visible behaviour with controlled providers/API | form error, loading/empty/error, pending button, accessible dialog |
| API adapter | Request/response/error contract mapping | Problem Details, DTO transform, tenant key, idempotency header |
| E2E | Critical real browser journey | login, deny access, booking create/cancel, role admin, tenant isolation |
| Visual | Appearance protection | design primitives/key screens at selected viewports |
| Non-functional | Accessibility, responsive, performance/security | focus, reduced motion, PWA offline/update, no token storage |

Test business risk—not a vanity coverage percentage. New/changed authentication, authorization UI, payment, booking availability, destructive action, tenant isolation, offline queue and complex workflow code requires stronger tests and manual review.

## Test design rules

- Test how users see and use the app: accessible roles, labels and visible text. Do not primarily test CSS classes, component instances or internal state.
- Every test is isolated: deterministic data/session/storage/clock/network and clean teardown. It passes alone and in parallel.
- Use stable test factories with clear tenant/user/permission ownership. Never test against production or mutable shared data.
- Mock external systems outside team control. Use controlled Spring API test environments for integration flows.
- Do not use arbitrary sleeps; wait for visible condition, controlled request, route or event.
- Test allowed and denied paths. Hidden action, deep link and server 403 must match the access contract.
- Snapshots are narrow/reviewed. Flaky tests are ticketed, owned and time-bound—never permanently bypassed.

## Mandatory scenario catalogue

Maintain test cases traceable to PRD acceptance criteria. Cover applicable cases:

- Public/authenticated/denied/not-found routes and safe deep-link return.
- Session bootstrap, expiry, one refresh retry, logout, disable/revocation, 401 versus 403 UX.
- Permission-aware navigation/actions, backend 403 after a UI allow, tenant cache isolation and tenant switch if enabled.
- Form labels, keyboard submit, validation, field errors, double-submit, conflict and uncertain-timeout recovery.
- Responsive key workflows, touch targets, zoom/reflow, keyboard/focus/screen-reader, contrast and reduced motion.
- API loading/empty/error/retry/rate-limit, pagination/filter/sort, stale/cancelled requests.
- XSS-safe rendering, storage/telemetry redaction, error boundary/chunk failure and CSP-safe integration.
- PWA/offline/update/push/queue only when enabled.

## Visual and browser checks

- Run visual tests in a fixed browser/OS/font environment with seeded data and deterministic time/motion. Review each baseline change.
- Test key routes at mobile and desktop widths, with long text, loading, empty, error and denied state; include dark/RTL if supported.
- E2E covers current supported Chromium, Firefox and WebKit where product support requires it. Manually check the agreed real-device matrix.

## CI quality gate

Every pull request runs:

```text
install from lockfile
-> type check
-> lint, import-boundary and unsafe-pattern checks
-> formatting verification
-> unit/component/API-adapter tests
-> production build and bundle budget
-> accessibility automation
-> targeted E2E / visual checks
-> dependency, security and license policy check
```

Main/release branches also run full critical E2E, supported-browser matrix, performance/PWA tests when applicable, and retain test/bundle/accessibility artifacts. Required failures block merge/release. Exceptions need linked reason, owner, expiry and follow-up.

## Release checklist

- Reviewed commit + reproducible lockfile build; production configuration correct and secret-free.
- API contract/version, feature flags, backend permission codes and tenant behaviour verified.
- Critical journeys pass in production-like environment with appropriate role/tenant accounts.
- Changed workflows pass keyboard/focus, responsive, loading/error/empty and WCAG checks.
- Error fallback, telemetry redaction, CSP/security headers and storage cleanup verified.
- Performance/Web Vitals evidence exists; new media/3D assets reviewed.
- If PWA enabled: manifest, service worker, cache, update, offline, logout and tenant-switch verified.
- Monitoring, alerts, rollback/support plan, release notes and temporary risk acceptances ready.

## Post-release

Monitor errors, Core Web Vitals, API/chunk-load failures, CSP violations, authentication/authorization UX failure and support tickets. Turn incidents into regression tests or playbook improvements.
