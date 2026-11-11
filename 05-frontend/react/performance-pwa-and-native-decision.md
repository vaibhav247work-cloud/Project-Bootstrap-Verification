# React Performance, PWA/Offline and Native-Mobile Decision Standard

## Default decision

Build one responsive React web application first. Add PWA capabilities only when installability, unreliable connectivity, offline use, push or app-like launch provides a named user/business value. Build a native mobile application only when a documented requirement cannot be met acceptably by responsive web/PWA.

Performance is a feature requirement, not cleanup work after design is complete.

## Performance budgets and measurement

Each project defines route-specific budgets based on its users, devices and network conditions. Start from these field-data targets at the 75th percentile:

| Metric | Default target | Meaning |
| --- | ---: | --- |
| LCP | <= 2.5 s | main content becomes visible |
| INP | <= 200 ms | interaction responsiveness |
| CLS | <= 0.1 | visual stability |

Also define maximum initial JS, route chunk, CSS, image/media, third-party and total transfer budgets for key routes. CI reports bundle-size regressions; a budget exception needs an ADR, owner and removal plan.

Measure both lab and real-user field data, segmented by route, device class, browser and connection where privacy-safe. Test low/mid-range Android-class hardware and throttled network—not only a developer laptop.

## Required implementation rules

- Split code by route/large feature with dynamic import/`React.lazy`; give every lazy boundary an accessible loading fallback and error boundary.
- Do not split tiny, always-co-loaded files only to create a request waterfall. Measure before and after.
- Load 3D engines, charts, rich editors, maps, PDF viewers, admin-only screens and large integrations only on the routes/features that need them.
- Remove unused dependencies, duplicate libraries/locales/icon packs and dead code. Analyse production bundles before major release.
- Defer/non-blocking-load non-critical analytics and third-party widgets after consent and after core interaction path is ready.
- Use modern responsive images (`srcset`/`sizes` or approved image component/CDN), suitable dimensions and compression; reserve dimensions to prevent CLS. Prefer SVG for simple icons, but sanitize untrusted SVG.
- Avoid rendering very large lists at once. Use server pagination/cursoring and virtualisation only when list size/measurement proves it useful; virtualised content retains keyboard/accessibility behaviour.
- Avoid long main-thread tasks, synchronous expensive calculations and unnecessary render cascades. Move genuinely heavy, non-UI computation to a Web Worker only with an explicit protocol, cancellation and fallback.
- Animate only compositor-friendly properties where possible and follow the reduced-motion standard. Decorative patterns/gradients/3D assets count against the route budget.
- Cache static versioned assets safely through CDN/deployment policy. HTML/app shell update and service-worker update rules must avoid serving incompatible old/new API contracts.

## PWA is an explicit capability

PWA is enabled only after the PRD answers:

1. Which user problem needs installability, offline reliability, background sync, push or device integration?
2. Which pages/data/actions work offline, read-only or queued? Which must fail clearly online-only?
3. What sensitive data may never enter service-worker/cache/IndexedDB storage?
4. How are version upgrade, cache cleanup, logout, tenant switch, account disable and revoked access handled?
5. Which browsers/platforms are supported and what is the ordinary-web fallback?

If these answers are absent, ship a responsive web app without a service worker. A service worker is optional enhancement, not a prerequisite for core functions.

## PWA implementation rules

- Serve only over HTTPS and provide a valid web app manifest, icons, name, start URL, display/theme configuration and tested install flow when installability is required.
- Use an approved service-worker/workbox strategy; do not write ad-hoc broad request interception.
- Static immutable build assets may use cache-first with versioned cleanup. Navigation/app-shell uses a controlled strategy that cannot trap users on an obsolete build.
- Public, non-sensitive GET content may use a documented network-first/stale-while-revalidate strategy only if staleness is safe.
- Authenticated API responses, permission/session data, sensitive PII, files and mutation responses are **not cached by default** in service-worker Cache Storage. Add each allowed offline data type by explicit design with tenant/user key, encryption/privacy decision, TTL, size limit, eviction and revoke/logout path.
- Offline writes/queues are permitted only for explicitly idempotent/reconcilable actions. Queue item includes user/tenant context, schema version, idempotency key, expiry, conflict strategy and user-visible sync status. Do not queue payment, security, irreversible approval or unknown-completion actions by default.
- Network loss gets a clear offline state; never show stale content as current without a timestamp/status.
- Update flow informs the user when a new app version is ready, applies it at a safe point, and handles old chunk/load failure through the global recovery path. Do not force-reload while an unsaved draft or critical mutation is active.
- Push notifications require a product purpose, explicit user permission at the correct moment, backend authorization/tenant targeting, opt-out, delivery/failure handling and privacy-safe payloads. Never place sensitive business/identity data in notification text by default.

## Native-mobile decision

Use this decision order:

| Need | Recommended default |
| --- | --- |
| Forms, dashboards, ERP, bookings, approvals, responsive portal | React web, optionally PWA |
| Installable launch, moderate offline read/cache, basic push where platform supports it | PWA after explicit offline design |
| Deep/background device work, advanced Bluetooth/NFC, continuous location, complex camera/media, platform-store requirement, highly reliable offline-first workflow | Assess React Native/native with ADR |

Before creating a native app, compare user value, device/API needs, offline requirement, platform coverage, accessibility, release/store operational cost, security storage, push/background differences, team skill and long-term shared-code strategy. "Mobile-friendly" alone is not a reason for a second codebase.

If native is approved, create a separate technology folder (for example `05-frontend/react-native/`) and define its own secure storage, platform permissions, release signing, deep-link, notification, offline and testing standards. Do not copy browser token/storage/service-worker assumptions to native.

## Required verification

- Bundle analysis and budget check on production build
- Lab and field Web Vitals for key routes, including low-end device/network scenario
- Route lazy-load success/failure/recovery, image/layout-shift and large-list tests
- PWA install, offline, cache, update, logout/tenant-switch/revocation and browser-fallback tests when PWA is enabled
- Native-vs-PWA ADR before a second mobile codebase is started
