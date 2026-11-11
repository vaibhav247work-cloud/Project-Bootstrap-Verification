# Default Technology Decisions

| Area | Default | Change only when |
|---|---|---|
| Architecture | Modular monolith | Independent deployment/scaling or strong domain ownership is proven necessary |
| Frontend | React + TypeScript + responsive-first | Another platform has a defined product requirement |
| Mobile | Responsive web | PWA benefits are clear, or native capabilities/store delivery are mandatory |
| Backend | Java + Spring Boot | Team, ecosystem, performance, delivery, or integration needs justify another choice |
| Database | MySQL | PostgreSQL features, existing client platform, or a documented operational reason requires it |
| API | REST + OpenAPI | Real-time/event/graph use case makes another contract more appropriate |
| Authentication | Central identity module | Client SSO/OIDC, B2C provider, or regulatory requirement needs an integration |

## Mobile decision guide

- **Responsive React web**: default for all products.
- **PWA**: add when installability, basic offline screens, push notifications, or app-like launch improves a validated user flow.
- **React Native**: evaluate only when native-store distribution or device features (reliable background work, Bluetooth, advanced camera/device integration, high-grade offline sync) are essential.
