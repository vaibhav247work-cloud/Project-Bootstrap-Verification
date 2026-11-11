# React Code Quality and Component Standard

## TypeScript and components

- Enable `strict`. Do not use `any`, `@ts-ignore` or unsafe assertions except a documented boundary.
- Types do not validate runtime data. Validate/map API, URL, storage and flag input at their boundary.
- Reuse approved generated API types/contracts; use separate view models where UI needs them.
- One component has one clear responsibility. Prefer composition and explicit props over universal components with many mode flags.
- Use semantic HTML: action is a `button`, navigation a link, and submission a `form`. Use stable item IDs—not array indexes—for mutable lists.
- Do not mutate props, state, cached responses or module-level objects. `memo`, `useMemo`, and `useCallback` need a real rendering reason, not default use.

## Styling/design system

Choose one approved styling method per project. Define reusable color, spacing, typography, breakpoint and focus tokens. UI supports applicable loading, empty, error, disabled, denied and long-content states. Do not rely on colour alone; keyboard focus and contrast are mandatory. Global CSS must be deliberately namespaced.

## File boundary and CI

Handwritten TS/TSX/CSS files are limited to **400 physical lines** for components and **300 physical lines** for pure utilities, hooks, schemas, and non-component modules. The absolute hard limit for any handwritten file is **600 lines**; a file approaching this limit must be split before it crosses it. Generated code is excluded and never hand-edited.

Split by responsibility — not by arbitrary line count. Good split examples:

```text
features/booking/
  BookingForm.tsx            ← form layout and field composition only
  booking-form.schema.ts     ← Zod/Yup validation schema
  useBookingDraft.ts         ← draft state and persistence logic
  BookingSubmitButton.tsx    ← isolated submit + loading/error UI

features/booking/api/
  booking-api.ts             ← DTO mapping and endpoint calls
  booking-api.types.ts       ← request/response types
```

Never create `BookingFormPart2.tsx`, `helpers2.ts`, or `utils.ts` dumping grounds to pass the limit.

CI runs type-check, lint, formatting, unit tests and production build. Tooling enforces hook correctness, unused code, accessibility baseline, unsafe HTML and import boundaries where supported. New warnings are fixed or ADR-approved.
