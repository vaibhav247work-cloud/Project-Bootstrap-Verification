# Frontend

Default frontend is **React + TypeScript**, responsive web first. A PWA or native-mobile application is a product/PRD decision, not an automatic extra application.

## Technology folders

```text
05-frontend/
  react/                       # React + TypeScript standards and implementation patterns
  README.md                     # cross-frontend scope and phase tracker
```

Future framework-specific standards, such as React Native or another frontend framework, receive their own folder. They must not be mixed into the React folder.

## Phase sub-steps

1. Application and module structure — complete (`react/react-architecture.md`)
2. Component, TypeScript, styling and design-system rules — complete (`react/frontend-code-quality.md`, `react/frontend-architecture-and-linter-rules.md`)
3. Routing, authentication and permission-aware UI — complete
4. API client, server state, local state and caching — complete
5. Forms, validation and user feedback — complete (`react/forms-validation-and-feedback.md`)
6. Data tables, grids, virtualization and bulk actions — complete (`react/data-table-and-grid-standard.md`)
7. Responsive design, accessibility, internationalization and modern visual UX — complete (`react/responsive-accessible-modern-ui.md`)
8. Frontend security, browser storage and error handling — complete (`react/browser-security-and-error-handling.md`)
9. Performance, PWA/offline and native-mobile decision rules — complete (`react/performance-pwa-and-native-decision.md`)
10. Frontend testing, quality gate and delivery checklist - complete (`react/testing-ci-and-release.md`)

Do not start a product UI until steps 1–7 are applicable and documented in its PRD/ADR. Steps 8 and native mobile are explicitly requirement-driven.
