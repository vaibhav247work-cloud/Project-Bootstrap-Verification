# Responsive, Reactive, Dynamic, Modern UI and Accessibility Standard

## Non-negotiable outcome

Every screen must be usable, readable, fast and accessible on small touch screens, desktop mouse/trackpad screens, keyboard-only navigation and assistive technology. Design mobile-first, then enhance for larger screens and capable devices.

"Modern" means clear hierarchy, intentional feedback, polished motion and consistent design tokens. It does **not** mean decoration that makes a product slower, harder to read, inaccessible or visually noisy.

## Responsive layout

1. Build the narrowest supported viewport first; no horizontal page scroll at 320 CSS px unless the content itself genuinely requires it (for example, a data table with an accessible alternate presentation).
2. Use fluid layout primitives: CSS Grid, Flexbox, intrinsic sizing, `minmax`, `clamp`, container queries where appropriate, responsive images and a small semantic breakpoint set. Do not design only for named devices.
3. Establish product design tokens for spacing, type scale, layout width, breakpoints, radius, elevation, z-index, colour and motion. Components consume tokens; screens do not invent arbitrary values.
4. Prefer progressive layout changes: side navigation may collapse to a drawer, multi-column forms become a readable single column, dense tables gain horizontal-scroll indication or card/detail alternative, and destructive controls remain visible and separated.
5. Reserve image/media dimensions to prevent layout shift. Use responsive image formats/sizes and lazy-load below-the-fold media.
6. Test at minimum: 320 px, 375/390 px, 768 px, 1024 px, 1440 px; portrait/landscape where relevant; browser zoom at 200%; and real touch device behaviour.
7. A responsive view changes layout—not meaning, required information, permission behaviour, validation, or core workflow.

## Reactive and dynamic UI

Reactive UI reflects current state promptly and predictably: request state, permission state, server changes, feature flags, viewport, input method and user preferences.

- Render explicit loading, empty, error, offline, stale, denied, disabled and success states. Never leave a blank area whose meaning is unclear.
- Use skeletons only when layout is known; they must resemble final content and not hide a slow/error state indefinitely. Use a spinner for short, local uncertain actions.
- Dynamic data/configuration is parsed and allow-listed. Server configuration may choose from approved layouts/components; it must not inject executable JavaScript, arbitrary CSS, raw HTML or unknown components.
- Update lists and summaries after mutations through the query-cache policy. Do not make the user refresh to see their own confirmed action.
- Real-time updates (WebSocket/SSE/polling) are a PRD decision. Define connection/auth lifecycle, tenant isolation, message schema/versioning, reconnect/backoff, ordering/conflict UX, background behaviour and fallback. Do not add real-time merely for appearance.
- Use feature flags as a controlled rollout mechanism, with loading/failure/default state, permission compatibility and removal date/owner. A feature flag never grants authorization.
- Respect user colour-scheme, reduced-motion, contrast and input-capability preferences when the product supports them.

## Visual system: colour, gradients and patterns

The design system defines semantic tokens rather than raw colours:

```text
surface / surface-raised / text / text-muted / border
primary / primary-hover / on-primary
success / warning / danger / info
focus-ring / disabled / overlay
```

- Text, icons, controls, charts and focus indicators meet the selected WCAG conformance target (default: WCAG 2.2 AA). Never use colour as the sole meaning of status, validation or selection.
- Support light theme by default; dark/high-contrast theme is a PRD/design-system decision but, when enabled, gets complete semantic tokens—not an automatic colour inversion.
- Gradients are permitted as subtle visual emphasis or non-critical background decoration. They must preserve text/icon contrast, never carry information alone, and have a solid-colour/fallback state where needed.
- Decorative patterns/textures must be lightweight CSS/SVG assets, low contrast and non-essential. Do not use large raster backgrounds, continuous animated gradients, or visual noise behind dense business data.
- Charts must not distinguish series by colour alone; use labels, shapes/patterns, tooltips and accessible data alternatives.

## Modern buttons, controls and feedback

Use an approved button component with semantic variants: primary (one main action per context), secondary, tertiary/ghost, destructive, icon-only and loading/disabled. A button triggers an action; a link navigates.

- Button labels begin with a clear verb where useful: `Save booking`, `Send invite`, `Delete employee`.
- An icon-only control requires an accessible name and visible tooltip on hover/focus when context is not obvious.
- Hover, pressed, focus-visible, disabled, loading and error/success states are all designed. Hover is an enhancement—core capability cannot depend on it.
- Disabled controls must have a reason available when it helps the user proceed; do not use disabled state to conceal authorization-sensitive capabilities where hiding is safer.
- Avoid fake buttons made from `div`/`span`; use native controls or a thoroughly accessible approved primitive.

## Pointer, touch, keyboard and screen optimisation

"Screen pointer optimisation" includes mouse, trackpad, touch, pen, keyboard and assistive input.

- Interactive targets are at least **24 × 24 CSS px** or have equivalent safe spacing; aim for **44 × 44 CSS px** for frequent touch actions and critical mobile controls.
- Keep adjacent destructive/small icon actions sufficiently separated. Never rely on hover alone for labels, menus, state or tooltips.
- Use pointer-capability media queries only as progressive enhancement; a device can switch between touch, mouse and keyboard.
- Drag-and-drop has an equivalent click/keyboard workflow. Gestures have visible controls; swipe/pinch is never the only way to act.
- Use `cursor: pointer` only for actual clickable controls; do not manipulate cursors to make non-interactive content appear interactive.
- Keyboard order follows visual/task order. Every action is reachable, visible focus is never removed, focus is managed after route/modal/error changes, and shortcuts do not conflict with typing/assistive technology.
- Tables, trees, comboboxes, dialogs, menus and custom widgets follow native HTML first or the relevant WAI-ARIA Authoring Practices pattern. Do not add ARIA roles to imitate native elements unnecessarily.

## Animation and motion

Motion communicates change, hierarchy or response; it is not decoration by default.

- Animate primarily `transform` and `opacity`; avoid layout/paint-heavy animation such as continuous `top`, `left`, width, height, box-shadow or large filter changes.
- Use short, purposeful durations and a consistent tokenised easing/duration scale. Never delay a user’s task merely to complete an animation.
- Honour `prefers-reduced-motion`: remove or replace non-essential motion, especially parallax, large scale/zoom, auto-rotating content, background movement, and lengthy page transitions.
- No auto-playing carousel, flashing, or looping attention animation without user control. Any essential movement has a non-motion equivalent.
- Pause/cancel animations cleanly when a component unmounts, tab is hidden, or a new user action supersedes it.

## 3D and rich visual effects

3D is optional and requires a product reason: e.g. spatial product configuration, educational visualization or meaningful interactive map. It is not a default dashboard decoration.

Before use, document an ADR covering user value, target devices, asset budget, fallback, performance measurement, reduced-motion path, keyboard/touch equivalent, screen-reader text/data alternative, battery/data impact and error recovery.

- Prefer CSS transforms or lightweight SVG for simple depth/tilt; load WebGL/Three.js/canvas only on the specific route and only when capability/performance permits.
- 3D/WebGL content is progressively enhanced. Core information and actions have an equivalent 2D semantic DOM path.
- Never put a required form, authentication, authorization, pricing, booking or destructive control exclusively in canvas/WebGL.
- Stop/limit rendering when hidden or inactive; cap quality on constrained devices and provide a user-visible disable/reduce-effects option if the feature is substantial.

## Accessibility baseline

- Default conformance target: WCAG 2.2 AA. Product-specific legal/contract requirements may demand more.
- Use landmarks (`header`, `nav`, `main`, `footer`), one logical page heading, semantic heading hierarchy, descriptive link text and a skip-to-main-content link.
- Every image has correct alternative treatment: meaningful image describes purpose; decorative image is ignored by assistive tech.
- Pages work at 200% zoom/reflow and without mouse, hover, sound, colour perception or motion.
- Modal dialogs trap focus appropriately, return focus to the invoker, offer Escape where safe and do not hide critical errors behind an inaccessible overlay.
- Do not announce every live update. Use concise `status`/alert patterns for meaningful dynamic changes only.
- Accessibility is verified with keyboard testing and at least one screen reader on representative routes; automated scanning supplements but never replaces manual testing.

## Internationalization, locale and RTL

Internationalization (i18n) is enabled when the PRD needs more than one language, country/region-specific formatting, or right-to-left (RTL) support. It is a product decision made before screens are built—not a last-minute text replacement exercise.

- All user-visible product strings come from a versioned locale catalog. Do not concatenate fragments such as `"Hello " + name`; use parameterized messages so translators can reorder words correctly.
- The catalog includes labels, validation/errors, empty states, tooltips, button states, dates, units, emails/notifications and accessibility text—not just page headings.
- Use the user/product locale for number, currency, percentage, plural, list and date/time presentation. Store and transport timestamps as UTC; display in the user/project-approved timezone with unambiguous format for business dates.
- Never hard-code a currency symbol, date order, decimal separator, first-day-of-week, plural rule or translated text inside a component.
- Text expansion is expected: layouts must accommodate longer translated labels without clipping, overlap, fixed-width button failures or reliance on text embedded in images.
- When RTL is enabled, use logical CSS properties (`margin-inline`, `padding-inline`, `inset-inline`, logical text alignment) and test every key workflow in RTL. Direction-sensitive icons (next/back, arrows, progress) mirror deliberately; universal icons do not.
- Language/locale changes invalidate formatted client views as needed but do not change the underlying server authority, tenant isolation or stored raw values.
- A project defines supported locales, fallback locale, language selection/persistence, translation ownership/review, release synchronization and missing-key behaviour in its PRD.

## Required design/implementation evidence

For every product release, record:

- token/design-system references and responsive layouts for key workflows;
- supported browsers/devices and tested viewport matrix;
- contrast, keyboard, focus, zoom/reflow, touch target and screen-reader checks;
- animation/reduced-motion checks;
- performance result for key routes; and
- ADR plus fallback verification for any 3D/rich effect or dynamic server-driven UI.
