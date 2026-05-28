# Smart Vehicle QR Assist — Complete Architecture & Technical Foundation

> **Document Type:** Architecture & Planning  
> **Version:** 1.0.0  
> **Status:** Draft — Pre-Development  
> **Classification:** Internal Technical Reference

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Complete Monorepo Structure](#2-complete-monorepo-structure)
3. [Software Architecture Diagrams](#3-software-architecture-diagrams)
4. [Database Planning & ERD](#4-database-planning--erd)
5. [API Architecture](#5-api-architecture)
6. [Security Architecture](#6-security-architecture)
7. [Feature Roadmap](#7-feature-roadmap)
8. [Technical Documentation](#8-technical-documentation)

---

## 1. Executive Summary

**Smart Vehicle QR Assist** is a privacy-first SaaS platform enabling anonymous, real-time communication between vehicle owners and anyone encountering an issue with their parked or stationary vehicle.

### Core Value Proposition

| Actor | Value |
|---|---|
| Vehicle Owner | Instant anonymous alerts without exposing personal info |
| Scanner / Reporter | Ability to help without revealing identity |
| Platform | Subscription SaaS + QR issuance + notification delivery |

### Privacy Contract

```
Scanner → [Report] → Platform → [Notification] → Owner
         ^                                ^
         No owner data exposed            No scanner data exposed
```

The platform acts as an **anonymous intermediary**. Zero personal data crosses the privacy boundary in either direction.

---

## 2. Complete Monorepo Structure

### Tool Choice: Turborepo + pnpm Workspaces

**Why Turborepo?** Incremental builds, remote caching, parallel task execution across apps and packages. `pnpm` workspaces for strict, performant dependency management.

```
smart-vehicle-qr-assist/
│
├── .github/                              # CI/CD and GitHub configuration
│   ├── workflows/
│   │   ├── ci.yml                        # Run lint, test, build on every PR
│   │   ├── cd-staging.yml                # Auto-deploy to staging on merge to develop
│   │   ├── cd-production.yml             # Deploy to production on release tag
│   │   ├── security-scan.yml             # Snyk/CodeQL security scanning
│   │   ├── db-migrate.yml                # Run Prisma migrations in CI
│   │   └── stale.yml                     # Auto-close stale issues/PRs
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   ├── feature_request.md
│   │   └── security_vulnerability.md
│   ├── PULL_REQUEST_TEMPLATE.md
│   ├── CODEOWNERS                        # Code ownership for review routing
│   └── dependabot.yml                    # Automated dependency updates
│
├── apps/
│   │
│   ├── web/                              # Next.js 15 — Primary Frontend
│   │   ├── public/
│   │   │   ├── icons/                    # App icons, favicon, OG images
│   │   │   ├── images/                   # Static images
│   │   │   └── manifest.json             # PWA manifest
│   │   ├── src/
│   │   │   ├── app/                      # Next.js App Router
│   │   │   │   ├── (auth)/               # Auth route group — no layout chrome
│   │   │   │   │   ├── login/
│   │   │   │   │   │   └── page.tsx
│   │   │   │   │   ├── register/
│   │   │   │   │   │   └── page.tsx
│   │   │   │   │   ├── verify-otp/
│   │   │   │   │   │   └── page.tsx
│   │   │   │   │   └── layout.tsx
│   │   │   │   ├── (dashboard)/          # Authenticated owner dashboard
│   │   │   │   │   ├── dashboard/
│   │   │   │   │   │   └── page.tsx      # Overview: vehicles, recent reports
│   │   │   │   │   ├── vehicles/
│   │   │   │   │   │   ├── page.tsx      # Vehicle list
│   │   │   │   │   │   ├── [id]/
│   │   │   │   │   │   │   └── page.tsx  # Vehicle detail + QR management
│   │   │   │   │   │   └── new/
│   │   │   │   │   │       └── page.tsx  # Add vehicle form
│   │   │   │   │   ├── reports/
│   │   │   │   │   │   ├── page.tsx      # All reports feed
│   │   │   │   │   │   └── [id]/
│   │   │   │   │   │       └── page.tsx  # Single report detail
│   │   │   │   │   ├── notifications/
│   │   │   │   │   │   └── page.tsx      # Notification preferences
│   │   │   │   │   ├── billing/
│   │   │   │   │   │   └── page.tsx      # Subscription & billing
│   │   │   │   │   ├── settings/
│   │   │   │   │   │   └── page.tsx      # Account settings
│   │   │   │   │   └── layout.tsx        # Dashboard shell with nav
│   │   │   │   ├── (public)/             # Public-facing marketing pages
│   │   │   │   │   ├── page.tsx          # Landing page
│   │   │   │   │   ├── pricing/
│   │   │   │   │   │   └── page.tsx
│   │   │   │   │   ├── how-it-works/
│   │   │   │   │   │   └── page.tsx
│   │   │   │   │   └── layout.tsx
│   │   │   │   ├── scan/                 # QR Scan flow — no auth required
│   │   │   │   │   └── [qrCode]/
│   │   │   │   │       ├── page.tsx      # Scanned landing page
│   │   │   │   │       └── success/
│   │   │   │   │           └── page.tsx  # Confirmation after report
│   │   │   │   ├── api/                  # Next.js API routes (thin proxies only)
│   │   │   │   │   └── auth/
│   │   │   │   │       └── [...nextauth]/
│   │   │   │   │           └── route.ts  # NextAuth OAuth callback handler
│   │   │   │   ├── globals.css
│   │   │   │   ├── layout.tsx            # Root layout
│   │   │   │   ├── not-found.tsx
│   │   │   │   └── error.tsx
│   │   │   ├── components/
│   │   │   │   ├── ui/                   # ShadCN generated components
│   │   │   │   │   ├── button.tsx
│   │   │   │   │   ├── card.tsx
│   │   │   │   │   ├── dialog.tsx
│   │   │   │   │   ├── form.tsx
│   │   │   │   │   ├── input.tsx
│   │   │   │   │   ├── select.tsx
│   │   │   │   │   ├── toast.tsx
│   │   │   │   │   └── ...
│   │   │   │   ├── layout/               # App-level layout components
│   │   │   │   │   ├── Navbar.tsx
│   │   │   │   │   ├── Sidebar.tsx
│   │   │   │   │   ├── Footer.tsx
│   │   │   │   │   └── MobileNav.tsx
│   │   │   │   ├── vehicles/
│   │   │   │   │   ├── VehicleCard.tsx
│   │   │   │   │   ├── VehicleForm.tsx
│   │   │   │   │   └── VehicleQRPanel.tsx
│   │   │   │   ├── reports/
│   │   │   │   │   ├── ReportCard.tsx
│   │   │   │   │   ├── ReportTimeline.tsx
│   │   │   │   │   └── ReportTypeIcon.tsx
│   │   │   │   ├── scan/
│   │   │   │   │   ├── ScanLanding.tsx   # What scanner sees after QR scan
│   │   │   │   │   ├── ReportTypeSelector.tsx
│   │   │   │   │   ├── ReportForm.tsx
│   │   │   │   │   ├── LocationCapture.tsx
│   │   │   │   │   └── PhotoUpload.tsx
│   │   │   │   ├── notifications/
│   │   │   │   │   ├── NotificationBell.tsx
│   │   │   │   │   └── NotificationItem.tsx
│   │   │   │   └── shared/
│   │   │   │       ├── QRCodeDisplay.tsx
│   │   │   │       ├── MapView.tsx
│   │   │   │       ├── LoadingSpinner.tsx
│   │   │   │       ├── ErrorBoundary.tsx
│   │   │   │       └── PrivacyBadge.tsx
│   │   │   ├── hooks/                    # Custom React hooks
│   │   │   │   ├── useAuth.ts
│   │   │   │   ├── useVehicles.ts
│   │   │   │   ├── useReports.ts
│   │   │   │   ├── useNotifications.ts
│   │   │   │   ├── useGeolocation.ts
│   │   │   │   ├── useFCM.ts             # Firebase Cloud Messaging hook
│   │   │   │   └── useQRScanner.ts
│   │   │   ├── store/                    # Zustand global state
│   │   │   │   ├── authStore.ts          # Auth state: user, tokens, session
│   │   │   │   ├── notificationStore.ts  # Unread count, notification list
│   │   │   │   ├── vehicleStore.ts       # Active vehicle, QR selection
│   │   │   │   └── uiStore.ts            # Modal state, sidebar, toasts
│   │   │   ├── lib/                      # Frontend utilities
│   │   │   │   ├── api/
│   │   │   │   │   ├── client.ts         # Axios instance with interceptors
│   │   │   │   │   ├── auth.api.ts
│   │   │   │   │   ├── vehicles.api.ts
│   │   │   │   │   ├── reports.api.ts
│   │   │   │   │   └── notifications.api.ts
│   │   │   │   ├── queries/              # React Query query/mutation defs
│   │   │   │   │   ├── auth.queries.ts
│   │   │   │   │   ├── vehicles.queries.ts
│   │   │   │   │   ├── reports.queries.ts
│   │   │   │   │   └── notifications.queries.ts
│   │   │   │   ├── firebase.ts           # FCM initialization
│   │   │   │   ├── maps.ts               # Google Maps loader
│   │   │   │   └── utils.ts
│   │   │   ├── types/                    # Frontend-specific types
│   │   │   │   └── index.ts
│   │   │   └── middleware.ts             # Next.js middleware: auth guards
│   │   ├── next.config.ts
│   │   ├── tailwind.config.ts
│   │   ├── tsconfig.json
│   │   ├── postcss.config.js
│   │   └── package.json
│   │
│   ├── api/                              # NestJS — Primary Backend
│   │   ├── src/
│   │   │   ├── main.ts                   # Bootstrap: Swagger, CORS, validation pipes
│   │   │   ├── app.module.ts             # Root module imports
│   │   │   ├── app.controller.ts         # Health check endpoint
│   │   │   │
│   │   │   ├── modules/                  # Feature modules
│   │   │   │   ├── auth/
│   │   │   │   │   ├── auth.module.ts
│   │   │   │   │   ├── auth.controller.ts
│   │   │   │   │   ├── auth.service.ts
│   │   │   │   │   ├── auth.repository.ts
│   │   │   │   │   ├── strategies/
│   │   │   │   │   │   ├── jwt.strategy.ts
│   │   │   │   │   │   ├── jwt-refresh.strategy.ts
│   │   │   │   │   │   ├── google.strategy.ts
│   │   │   │   │   │   └── local.strategy.ts
│   │   │   │   │   ├── guards/
│   │   │   │   │   │   ├── jwt-auth.guard.ts
│   │   │   │   │   │   ├── google-auth.guard.ts
│   │   │   │   │   │   └── roles.guard.ts
│   │   │   │   │   ├── dto/
│   │   │   │   │   │   ├── register.dto.ts
│   │   │   │   │   │   ├── login.dto.ts
│   │   │   │   │   │   ├── verify-otp.dto.ts
│   │   │   │   │   │   ├── refresh-token.dto.ts
│   │   │   │   │   │   └── forgot-password.dto.ts
│   │   │   │   │   └── decorators/
│   │   │   │   │       ├── current-user.decorator.ts
│   │   │   │   │       └── roles.decorator.ts
│   │   │   │   │
│   │   │   │   ├── users/
│   │   │   │   │   ├── users.module.ts
│   │   │   │   │   ├── users.controller.ts
│   │   │   │   │   ├── users.service.ts
│   │   │   │   │   ├── users.repository.ts
│   │   │   │   │   └── dto/
│   │   │   │   │       ├── update-user.dto.ts
│   │   │   │   │       ├── update-notification-prefs.dto.ts
│   │   │   │   │       └── user-response.dto.ts
│   │   │   │   │
│   │   │   │   ├── vehicles/
│   │   │   │   │   ├── vehicles.module.ts
│   │   │   │   │   ├── vehicles.controller.ts
│   │   │   │   │   ├── vehicles.service.ts
│   │   │   │   │   ├── vehicles.repository.ts
│   │   │   │   │   └── dto/
│   │   │   │   │       ├── create-vehicle.dto.ts
│   │   │   │   │       ├── update-vehicle.dto.ts
│   │   │   │   │       └── vehicle-response.dto.ts
│   │   │   │   │
│   │   │   │   ├── qr-codes/
│   │   │   │   │   ├── qr-codes.module.ts
│   │   │   │   │   ├── qr-codes.controller.ts
│   │   │   │   │   ├── qr-codes.service.ts
│   │   │   │   │   ├── qr-codes.repository.ts
│   │   │   │   │   └── dto/
│   │   │   │   │       ├── generate-qr.dto.ts
│   │   │   │   │       └── qr-response.dto.ts
│   │   │   │   │
│   │   │   │   ├── reports/
│   │   │   │   │   ├── reports.module.ts
│   │   │   │   │   ├── reports.controller.ts
│   │   │   │   │   ├── reports.service.ts
│   │   │   │   │   ├── reports.repository.ts
│   │   │   │   │   └── dto/
│   │   │   │   │       ├── create-report.dto.ts
│   │   │   │   │       ├── update-report-status.dto.ts
│   │   │   │   │       └── report-response.dto.ts
│   │   │   │   │
│   │   │   │   ├── notifications/
│   │   │   │   │   ├── notifications.module.ts
│   │   │   │   │   ├── notifications.controller.ts
│   │   │   │   │   ├── notifications.service.ts
│   │   │   │   │   ├── notifications.repository.ts
│   │   │   │   │   ├── providers/
│   │   │   │   │   │   ├── fcm.provider.ts
│   │   │   │   │   │   ├── email.provider.ts
│   │   │   │   │   │   ├── sms.provider.ts
│   │   │   │   │   │   └── notification-factory.ts
│   │   │   │   │   └── dto/
│   │   │   │   │       ├── send-notification.dto.ts
│   │   │   │   │       └── notification-response.dto.ts
│   │   │   │   │
│   │   │   │   ├── subscriptions/
│   │   │   │   │   ├── subscriptions.module.ts
│   │   │   │   │   ├── subscriptions.controller.ts
│   │   │   │   │   ├── subscriptions.service.ts
│   │   │   │   │   ├── subscriptions.repository.ts
│   │   │   │   │   └── dto/
│   │   │   │   │       ├── create-subscription.dto.ts
│   │   │   │   │       └── subscription-response.dto.ts
│   │   │   │   │
│   │   │   │   ├── storage/
│   │   │   │   │   ├── storage.module.ts
│   │   │   │   │   ├── storage.service.ts    # Cloudinary wrapper
│   │   │   │   │   └── storage.controller.ts # Signed upload URL generation
│   │   │   │   │
│   │   │   │   └── admin/
│   │   │   │       ├── admin.module.ts
│   │   │   │       ├── admin.controller.ts
│   │   │   │       ├── admin.service.ts
│   │   │   │       └── dto/
│   │   │   │           ├── admin-stats.dto.ts
│   │   │   │           └── moderation.dto.ts
│   │   │   │
│   │   │   ├── common/                   # Shared across all modules
│   │   │   │   ├── decorators/
│   │   │   │   │   ├── api-response.decorator.ts
│   │   │   │   │   ├── public.decorator.ts
│   │   │   │   │   └── throttle.decorator.ts
│   │   │   │   ├── filters/
│   │   │   │   │   ├── global-exception.filter.ts
│   │   │   │   │   ├── prisma-exception.filter.ts
│   │   │   │   │   └── validation-exception.filter.ts
│   │   │   │   ├── guards/
│   │   │   │   │   ├── throttle.guard.ts
│   │   │   │   │   └── captcha.guard.ts
│   │   │   │   ├── interceptors/
│   │   │   │   │   ├── logging.interceptor.ts
│   │   │   │   │   ├── transform-response.interceptor.ts
│   │   │   │   │   └── audit.interceptor.ts
│   │   │   │   ├── middlewares/
│   │   │   │   │   ├── logger.middleware.ts
│   │   │   │   │   └── correlation-id.middleware.ts
│   │   │   │   ├── pipes/
│   │   │   │   │   ├── validation.pipe.ts
│   │   │   │   │   └── parse-objectid.pipe.ts
│   │   │   │   └── utils/
│   │   │   │       ├── crypto.util.ts
│   │   │   │       ├── qr-generator.util.ts
│   │   │   │       └── pagination.util.ts
│   │   │   │
│   │   │   └── config/
│   │   │       ├── app.config.ts
│   │   │       ├── database.config.ts
│   │   │       ├── jwt.config.ts
│   │   │       ├── firebase.config.ts
│   │   │       ├── cloudinary.config.ts
│   │   │       ├── twilio.config.ts
│   │   │       ├── email.config.ts
│   │   │       └── redis.config.ts
│   │   │
│   │   ├── prisma/
│   │   │   ├── schema.prisma             # Single source of truth for DB schema
│   │   │   ├── seed.ts                   # Dev seed data
│   │   │   └── migrations/               # Auto-generated migration files
│   │   │
│   │   ├── test/
│   │   │   ├── app.e2e-spec.ts
│   │   │   ├── auth.e2e-spec.ts
│   │   │   ├── vehicles.e2e-spec.ts
│   │   │   ├── reports.e2e-spec.ts
│   │   │   └── jest-e2e.json
│   │   │
│   │   ├── .env.example
│   │   ├── nest-cli.json
│   │   ├── tsconfig.json
│   │   ├── tsconfig.build.json
│   │   └── package.json
│   │
│   └── admin/                            # Admin Dashboard (separate Next.js app)
│       ├── src/
│       │   ├── app/
│       │   │   ├── dashboard/
│       │   │   │   └── page.tsx          # Platform-wide metrics
│       │   │   ├── users/
│       │   │   │   └── page.tsx
│       │   │   ├── reports/
│       │   │   │   └── page.tsx          # Moderation queue
│       │   │   └── layout.tsx
│       │   └── ...
│       └── package.json
│
├── packages/                             # Shared code across apps
│   │
│   ├── types/                            # Shared TypeScript types
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   ├── user.types.ts
│   │   │   ├── vehicle.types.ts
│   │   │   ├── report.types.ts
│   │   │   ├── notification.types.ts
│   │   │   ├── qr.types.ts
│   │   │   ├── subscription.types.ts
│   │   │   └── api-response.types.ts
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   ├── validators/                       # Shared Zod validation schemas
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   ├── auth.schema.ts
│   │   │   ├── vehicle.schema.ts
│   │   │   ├── report.schema.ts
│   │   │   └── notification.schema.ts
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   ├── config/                           # Shared config constants
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   ├── report-types.ts           # Enum: WRONG_PARK, BLOCKED, LIGHTS_ON, etc.
│   │   │   ├── notification-channels.ts
│   │   │   ├── subscription-plans.ts
│   │   │   └── error-codes.ts
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   ├── ui/                               # Shared design system (if admin shares components)
│   │   ├── src/
│   │   │   └── components/
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   └── logger/                           # Shared structured logging (Pino)
│       ├── src/
│       │   └── index.ts
│       ├── tsconfig.json
│       └── package.json
│
├── services/                             # External service configs/wrappers
│   ├── firebase/
│   │   ├── firebase-admin.config.ts      # Service account config
│   │   └── fcm-templates/
│   │       ├── wrong-parking.json
│   │       ├── emergency.json
│   │       └── blocked-vehicle.json
│   │
│   ├── email/
│   │   ├── templates/                    # HTML email templates
│   │   │   ├── report-received.html
│   │   │   ├── welcome.html
│   │   │   ├── otp-verification.html
│   │   │   └── subscription-confirm.html
│   │   └── email.config.ts
│   │
│   ├── sms/
│   │   ├── twilio.config.ts
│   │   └── templates/
│   │       ├── otp.ts
│   │       └── report-alert.ts
│   │
│   └── redis/
│       └── redis.config.ts               # Rate limiting, session, OTP caching
│
├── infrastructure/                       # DevOps and deployment configs
│   ├── docker/
│   │   ├── Dockerfile.api                # NestJS production Dockerfile
│   │   ├── Dockerfile.web                # Next.js production Dockerfile
│   │   ├── Dockerfile.admin
│   │   └── docker-compose.yml            # Full local stack
│   │
│   ├── kubernetes/                       # K8s manifests (future scale)
│   │   ├── api-deployment.yaml
│   │   ├── web-deployment.yaml
│   │   ├── postgres-statefulset.yaml
│   │   ├── redis-deployment.yaml
│   │   ├── ingress.yaml
│   │   └── secrets.yaml
│   │
│   ├── terraform/                        # IaC for cloud resources
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── modules/
│   │   │   ├── rds/
│   │   │   ├── elasticache/
│   │   │   └── ecs/
│   │   └── environments/
│   │       ├── staging/
│   │       └── production/
│   │
│   └── nginx/
│       └── nginx.conf                    # Reverse proxy + SSL termination
│
├── docs/                                 # All documentation
│   ├── architecture/
│   │   ├── system-overview.md
│   │   ├── database-design.md
│   │   ├── api-design.md
│   │   ├── security.md
│   │   └── adr/                          # Architecture Decision Records
│   │       ├── ADR-001-monorepo.md
│   │       ├── ADR-002-nestjs-backend.md
│   │       ├── ADR-003-prisma-orm.md
│   │       ├── ADR-004-privacy-model.md
│   │       └── ADR-005-qr-strategy.md
│   ├── api/
│   │   └── openapi.yaml                  # OpenAPI 3.0 spec
│   ├── guides/
│   │   ├── local-setup.md
│   │   ├── deployment.md
│   │   ├── contributing.md
│   │   └── environment-variables.md
│   └── runbooks/
│       ├── incident-response.md
│       ├── db-migration.md
│       └── rollback.md
│
├── tests/                                # Cross-app test utilities
│   ├── fixtures/
│   │   ├── users.fixture.ts
│   │   ├── vehicles.fixture.ts
│   │   └── reports.fixture.ts
│   ├── helpers/
│   │   ├── db-setup.ts
│   │   └── auth-helper.ts
│   └── jest.config.base.ts
│
├── scripts/                              # Developer utility scripts
│   ├── setup.sh                          # Initial dev environment setup
│   ├── seed-db.sh
│   ├── generate-qr.ts                    # QR generation CLI
│   └── clean.sh
│
├── .env.example                          # Template for all environment variables
├── .eslintrc.js                          # Shared ESLint config
├── .prettierrc                           # Shared Prettier config
├── .gitignore
├── .nvmrc                                # Node version pin
├── turbo.json                            # Turborepo pipeline config
├── pnpm-workspace.yaml                   # pnpm workspace definition
├── package.json                          # Root package.json
└── README.md
```

---

## 3. Software Architecture Diagrams

### 3.1 System Architecture Diagram

```mermaid
graph TB
    subgraph CLIENT["Client Layer"]
        WEB["Next.js 15 Web App<br/>(Owner Dashboard)"]
        MOBILE["Mobile Browser<br/>(Scanner / Reporter)"]
        ADMIN_APP["Admin Dashboard<br/>(Next.js)"]
    end

    subgraph CDN["Edge / CDN"]
        CF["Cloudflare CDN<br/>SSL Termination<br/>DDoS Protection"]
    end

    subgraph GATEWAY["API Gateway"]
        NGINX["NGINX<br/>Reverse Proxy<br/>Rate Limiting"]
    end

    subgraph BACKEND["Backend Services"]
        API["NestJS API Server<br/>REST + WebSocket"]
        REDIS["Redis<br/>Session Cache<br/>Rate Limit<br/>OTP Store"]
    end

    subgraph DATABASE["Data Layer"]
        PG[("PostgreSQL<br/>Primary DB")]
        PG_READ[("PostgreSQL<br/>Read Replica")]
    end

    subgraph STORAGE["Storage"]
        CLOUDINARY["Cloudinary<br/>Images + QR Assets"]
    end

    subgraph NOTIFICATIONS["Notification Services"]
        FCM["Firebase FCM<br/>Push Notifications"]
        SENDGRID["SendGrid / SES<br/>Email"]
        TWILIO["Twilio<br/>SMS / OTP"]
    end

    subgraph EXTERNAL["External APIs"]
        GOOGLE_AUTH["Google OAuth 2.0"]
        GOOGLE_MAPS["Google Maps API"]
    end

    subgraph OBSERVABILITY["Observability"]
        LOGS["Log Aggregation<br/>(Loki / CloudWatch)"]
        METRICS["Metrics<br/>(Prometheus + Grafana)"]
        TRACES["Tracing<br/>(OpenTelemetry)"]
    end

    WEB -->|HTTPS| CF
    MOBILE -->|HTTPS| CF
    ADMIN_APP -->|HTTPS| CF
    CF --> NGINX
    NGINX --> API
    API <--> REDIS
    API -->|Read/Write| PG
    API -->|Read Only| PG_READ
    API --> CLOUDINARY
    API --> FCM
    API --> SENDGRID
    API --> TWILIO
    API --> GOOGLE_AUTH
    API --> GOOGLE_MAPS
    API --> LOGS
    API --> METRICS
    API --> TRACES
```

### 3.2 Component Diagram

```mermaid
graph LR
    subgraph NEXT["Next.js Frontend"]
        AUTH_PAGES["Auth Pages<br/>Login / Register / OTP"]
        DASHBOARD["Owner Dashboard<br/>Vehicles + Reports"]
        SCAN_FLOW["Scan Flow<br/>Reporter Interface"]

        subgraph STATE["State Layer"]
            ZUSTAND["Zustand Stores<br/>auth / vehicle / notifications / ui"]
            RQ["React Query<br/>Server State Cache"]
        end

        subgraph HTTP["HTTP Layer"]
            AXIOS["Axios Client<br/>+ Interceptors<br/>Auto Token Refresh"]
        end
    end

    subgraph NEST["NestJS Backend"]
        subgraph MODULES["Feature Modules"]
            AUTH_MOD["Auth Module"]
            USERS_MOD["Users Module"]
            VEHICLES_MOD["Vehicles Module"]
            QR_MOD["QR Codes Module"]
            REPORTS_MOD["Reports Module"]
            NOTIF_MOD["Notifications Module"]
            SUB_MOD["Subscriptions Module"]
            STORAGE_MOD["Storage Module"]
            ADMIN_MOD["Admin Module"]
        end

        subgraph CROSS["Cross-Cutting"]
            GUARDS["Guards<br/>JWT / Roles / Throttle / Captcha"]
            FILTERS["Exception Filters<br/>Global / Prisma / Validation"]
            INTERCEPTORS["Interceptors<br/>Logging / Transform / Audit"]
            PIPES["Pipes<br/>Validation / Sanitization"]
        end
    end

    DASHBOARD --> RQ
    SCAN_FLOW --> RQ
    RQ --> AXIOS
    AXIOS --> GUARDS
    GUARDS --> MODULES
    MODULES --> CROSS
```

### 3.3 QR Scan & Report Submission Flow

```mermaid
sequenceDiagram
    actor Scanner as Scanner (Anonymous)
    participant QR as QR Code (Physical)
    participant WEB as Web App (/scan/:code)
    participant API as NestJS API
    participant DB as PostgreSQL
    participant CAPTCHA as CAPTCHA Service
    participant NOTIF as Notification Service
    participant OWNER as Vehicle Owner

    Scanner->>QR: Scans QR Code (Camera)
    QR-->>Scanner: Redirects to https://app.com/scan/{unique_qr_token}

    Scanner->>WEB: GET /scan/{unique_qr_token}
    WEB->>API: GET /api/v1/qr/{token}/validate
    API->>DB: Lookup QR token → vehicle (active, not expired)
    DB-->>API: QR valid, vehicle info (masked: make/color only)
    API-->>WEB: { vehicle: { make, color }, reportTypes: [...] }
    WEB-->>Scanner: Show report form (no owner info visible)

    Scanner->>WEB: Selects report type, adds description, photo, grants location
    WEB->>CAPTCHA: Verify CAPTCHA token
    CAPTCHA-->>WEB: Valid

    WEB->>API: POST /api/v1/reports\n{ qrToken, type, description, photos, location, captchaToken }
    API->>API: Validate & sanitize input
    API->>API: Strip any PII from freetext (regex / ML scrubber)
    API->>CLOUDINARY: Upload photo(s) → signed URL
    CLOUDINARY-->>API: Secure photo URLs

    API->>DB: INSERT report (linked to vehicle, not to scanner)
    DB-->>API: Report created (reportId)

    API->>NOTIF: Trigger notification for vehicle owner
    NOTIF->>NOTIF: Fetch owner preferences (push / email / SMS)
    par Push
        NOTIF->>Owner: FCM Push Notification
    and Email
        NOTIF->>Owner: Email Alert
    and SMS
        NOTIF->>Owner: SMS via Twilio
    end

    API-->>WEB: { success: true, reportId, message: "Owner notified" }
    WEB-->>Scanner: Confirmation screen (no owner data shown)
```

### 3.4 Authentication Flow

```mermaid
sequenceDiagram
    actor User
    participant WEB as Next.js Frontend
    participant API as NestJS API
    participant REDIS as Redis
    participant DB as PostgreSQL
    participant GOOGLE as Google OAuth
    participant TWILIO as Twilio SMS

    Note over User,TWILIO: Path A — Email + Password

    User->>WEB: Login (email + password)
    WEB->>API: POST /auth/login
    API->>DB: Find user by email
    DB-->>API: User record
    API->>API: bcrypt.compare(password, hash)
    API->>API: Generate access_token (15m) + refresh_token (7d)
    API->>REDIS: Store refresh token hash (key: userId)
    API-->>WEB: { access_token, refresh_token, user }
    WEB->>WEB: Store access_token in memory (Zustand)\nStore refresh_token in httpOnly cookie

    Note over User,TWILIO: Path B — Google OAuth

    User->>WEB: "Continue with Google"
    WEB->>GOOGLE: OAuth redirect
    GOOGLE-->>WEB: Authorization code
    WEB->>API: POST /auth/google/callback { code }
    API->>GOOGLE: Exchange code → profile
    API->>DB: Upsert user (findOrCreate by email)
    API->>API: Issue access + refresh tokens
    API-->>WEB: { access_token, refresh_token, user }

    Note over User,TWILIO: Path C — OTP (Phone)

    User->>WEB: Enter phone number
    WEB->>API: POST /auth/otp/send { phone }
    API->>API: Generate 6-digit OTP
    API->>REDIS: SET otp:{phone} = hash(otp) EX 300
    API->>TWILIO: Send SMS OTP
    User->>WEB: Enter OTP
    WEB->>API: POST /auth/otp/verify { phone, otp }
    API->>REDIS: Validate hash(otp) match
    API->>DB: Upsert user by phone
    API-->>WEB: { access_token, refresh_token, user }

    Note over User,TWILIO: Token Refresh Flow

    WEB->>API: Any request → 401 Unauthorized
    WEB->>API: POST /auth/refresh { refresh_token (from cookie) }
    API->>REDIS: Validate refresh token hash
    API->>API: Rotate tokens (issue new pair)
    API->>REDIS: Invalidate old refresh token
    API-->>WEB: { new_access_token, new_refresh_token }
```

### 3.5 Notification Flow

```mermaid
flowchart TD
    REPORT[Report Created in DB]
    
    REPORT --> FETCH[Fetch Owner Preferences]
    FETCH --> PREFS{Notification Channels}
    
    PREFS -->|push_enabled| FCM_CHECK{FCM Token Exists?}
    PREFS -->|email_enabled| EMAIL[Queue Email Job]
    PREFS -->|sms_enabled| SMS[Queue SMS Job]
    
    FCM_CHECK -->|Yes| FCM[Send FCM Push]
    FCM_CHECK -->|No| FCM_SKIP[Skip Push]
    
    FCM -->|Success| FCM_LOG[Log: Delivered]
    FCM -->|Failed - Token Invalid| FCM_CLEAN[Remove Stale Token from DB]
    FCM -->|Failed - Retry| FCM_RETRY[Retry Queue x3]
    
    EMAIL --> TEMPLATE[Render Email Template\nreport-type specific]
    TEMPLATE --> SENDGRID[SendGrid / SES Send]
    SENDGRID -->|Success| EMAIL_LOG[Log: Delivered]
    SENDGRID -->|Failed| EMAIL_RETRY[Retry Queue x3]
    
    SMS --> TWILIO[Twilio Send SMS]
    TWILIO -->|Success| SMS_LOG[Log: Delivered]
    TWILIO -->|Failed| SMS_RETRY[Retry Queue x3]
    
    FCM_LOG --> AUDIT[Audit Log: notification_id, channel, status, timestamp]
    EMAIL_LOG --> AUDIT
    SMS_LOG --> AUDIT
    
    AUDIT --> NOTIF_RECORD[Update Notification Record in DB]
```

### 3.6 Request Flow Diagram (Standard Authenticated Request)

```mermaid
sequenceDiagram
    participant CLIENT as Client (Browser)
    participant NGINX as NGINX
    participant THROTTLE as ThrottleGuard (Redis)
    participant JWT as JWTAuthGuard
    participant ROLES as RolesGuard
    participant PIPE as ValidationPipe
    participant CTRL as Controller
    participant SVC as Service
    participant REPO as Repository
    participant DB as PostgreSQL
    participant INTERCEPT as ResponseInterceptor + AuditInterceptor

    CLIENT->>NGINX: HTTPS Request + Bearer Token
    NGINX->>NGINX: Rate limit by IP (nginx layer)
    NGINX->>THROTTLE: Forward to NestJS
    THROTTLE->>THROTTLE: Check Redis: requests per user per window
    THROTTLE-->>CLIENT: 429 Too Many Requests (if exceeded)
    THROTTLE->>JWT: Pass through
    JWT->>JWT: Verify access_token signature + expiry
    JWT-->>CLIENT: 401 Unauthorized (if invalid)
    JWT->>ROLES: Attach user to request
    ROLES->>ROLES: Check user.roles vs @Roles() decorator
    ROLES-->>CLIENT: 403 Forbidden (if insufficient role)
    ROLES->>PIPE: Authorized request
    PIPE->>PIPE: class-validator: validate DTO shape
    PIPE->>PIPE: class-transformer: sanitize/transform
    PIPE-->>CLIENT: 400 Bad Request (if invalid)
    PIPE->>CTRL: Clean, validated request
    CTRL->>SVC: Business logic call
    SVC->>REPO: Data access call
    REPO->>DB: Prisma query
    DB-->>REPO: Data
    REPO-->>SVC: Typed response
    SVC-->>CTRL: Business result
    CTRL->>INTERCEPT: Raw response
    INTERCEPT->>INTERCEPT: Transform to { data, meta, status }
    INTERCEPT->>INTERCEPT: Write audit log
    INTERCEPT-->>CLIENT: Standardized JSON response
```

---

## 4. Database Planning & ERD

### 4.1 Entity Relationship Diagram

```mermaid
erDiagram
    USERS {
        uuid id PK
        string email UK
        string phone UK
        string password_hash
        string name
        string avatar_url
        string google_id UK
        enum role "OWNER | ADMIN | SUPER_ADMIN"
        boolean is_verified
        boolean is_active
        timestamp email_verified_at
        timestamp phone_verified_at
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }

    USER_NOTIFICATION_PREFERENCES {
        uuid id PK
        uuid user_id FK
        boolean push_enabled
        boolean email_enabled
        boolean sms_enabled
        boolean emergency_push
        boolean emergency_email
        boolean emergency_sms
        string[] fcm_tokens
        timestamp updated_at
    }

    SESSIONS {
        uuid id PK
        uuid user_id FK
        string refresh_token_hash
        string ip_address
        string user_agent
        string device_fingerprint
        boolean is_revoked
        timestamp expires_at
        timestamp created_at
        timestamp last_used_at
    }

    ROLES {
        uuid id PK
        string name UK
        string description
        timestamp created_at
    }

    PERMISSIONS {
        uuid id PK
        string resource
        string action
        string description
        timestamp created_at
    }

    ROLE_PERMISSIONS {
        uuid role_id FK
        uuid permission_id FK
    }

    USER_ROLES {
        uuid user_id FK
        uuid role_id FK
        timestamp assigned_at
    }

    VEHICLES {
        uuid id PK
        uuid owner_id FK
        string license_plate
        string make
        string model
        string year
        string color
        string vin
        string image_url
        enum status "ACTIVE | INACTIVE | SOLD"
        timestamp created_at
        timestamp updated_at
        timestamp deleted_at
    }

    QR_CODES {
        uuid id PK
        uuid vehicle_id FK
        string token UK
        string qr_image_url
        string short_url
        enum status "ACTIVE | INACTIVE | EXPIRED | REPLACED"
        integer scan_count
        timestamp last_scanned_at
        timestamp expires_at
        timestamp created_at
        timestamp updated_at
    }

    REPORTS {
        uuid id PK
        uuid qr_code_id FK
        uuid vehicle_id FK
        string anonymous_session_id
        enum type "WRONG_PARKING | BLOCKED_VEHICLE | LIGHTS_ON | ACCIDENT | EMERGENCY | OTHER"
        enum severity "LOW | MEDIUM | HIGH | CRITICAL"
        string description
        string[] photo_urls
        float latitude
        float longitude
        string formatted_address
        enum status "PENDING | NOTIFIED | ACKNOWLEDGED | RESOLVED | EXPIRED"
        boolean is_flagged
        timestamp owner_acknowledged_at
        timestamp resolved_at
        timestamp created_at
        timestamp updated_at
    }

    NOTIFICATIONS {
        uuid id PK
        uuid user_id FK
        uuid report_id FK
        enum channel "PUSH | EMAIL | SMS"
        enum status "PENDING | SENT | DELIVERED | FAILED | RETRYING"
        string subject
        string body
        integer retry_count
        string error_message
        timestamp sent_at
        timestamp delivered_at
        timestamp created_at
    }

    SUBSCRIPTIONS {
        uuid id PK
        uuid user_id FK
        string plan_id
        enum plan_type "FREE | BASIC | PRO | ENTERPRISE"
        enum status "ACTIVE | CANCELLED | EXPIRED | PAST_DUE"
        integer max_vehicles
        integer max_qr_codes_per_vehicle
        boolean sms_notifications
        boolean priority_support
        string payment_provider
        string payment_subscription_id
        timestamp current_period_start
        timestamp current_period_end
        timestamp cancelled_at
        timestamp created_at
        timestamp updated_at
    }

    AUDIT_LOGS {
        uuid id PK
        uuid user_id FK
        string action
        string resource_type
        string resource_id
        jsonb before_state
        jsonb after_state
        string ip_address
        string user_agent
        string correlation_id
        timestamp created_at
    }

    OTP_VERIFICATIONS {
        uuid id PK
        string recipient
        enum type "EMAIL | PHONE"
        enum purpose "REGISTER | LOGIN | RESET_PASSWORD | VERIFY"
        string code_hash
        integer attempts
        boolean is_used
        timestamp expires_at
        timestamp created_at
    }

    USERS ||--o{ VEHICLES : "owns"
    USERS ||--o| USER_NOTIFICATION_PREFERENCES : "has"
    USERS ||--o{ SESSIONS : "has"
    USERS ||--o{ SUBSCRIPTIONS : "subscribes"
    USERS ||--o{ AUDIT_LOGS : "generates"
    USERS ||--o{ USER_ROLES : "assigned"
    ROLES ||--o{ USER_ROLES : "granted to"
    ROLES ||--o{ ROLE_PERMISSIONS : "grants"
    PERMISSIONS ||--o{ ROLE_PERMISSIONS : "in"
    VEHICLES ||--o{ QR_CODES : "has"
    QR_CODES ||--o{ REPORTS : "receives"
    VEHICLES ||--o{ REPORTS : "subject of"
    REPORTS ||--o{ NOTIFICATIONS : "triggers"
    USERS ||--o{ NOTIFICATIONS : "receives"
```

### 4.2 Table Relationships Summary

| Table | Relates To | Relationship | Purpose |
|---|---|---|---|
| `users` | `vehicles` | One-to-Many | A user owns multiple vehicles |
| `users` | `sessions` | One-to-Many | Multi-device session management |
| `users` | `subscriptions` | One-to-Many | Subscription history per user |
| `users` | `user_roles` | Many-to-Many via join | RBAC role assignment |
| `roles` | `permissions` | Many-to-Many via join | Fine-grained RBAC |
| `vehicles` | `qr_codes` | One-to-Many | Multiple QR generations per vehicle |
| `qr_codes` | `reports` | One-to-Many | A QR receives many reports over time |
| `reports` | `notifications` | One-to-Many | One report triggers multiple notifications per channel |
| `users` | `notification_preferences` | One-to-One | Per-user notification settings |

### 4.3 Critical Design Decisions

**Privacy Isolation:**
The `reports` table deliberately has **no foreign key to a user/scanner**. The `anonymous_session_id` is a client-generated UUID — not linked to any user account. This is the core privacy guarantee.

**QR Token Design:**
QR tokens are UUID v4 generated server-side. They are **not** the vehicle's license plate, VIN, or any reversible identifier. A compromised token only exposes "there is a vehicle" — not whose.

**Soft Deletes:**
`users` and `vehicles` use `deleted_at` (soft delete) to preserve referential integrity for reports and audit logs, while honoring GDPR erasure requests through a separate anonymization job.

**Subscription Limits Enforcement:**
`subscriptions.max_vehicles` and `max_qr_codes_per_vehicle` are enforced at the service layer, not the database constraint layer, for flexibility.

---

## 5. API Architecture

### 5.1 Versioning Strategy

All routes are prefixed with `/api/v1/`. Breaking changes require a new version prefix (`/api/v2/`). Non-breaking additions (new optional fields, new endpoints) do not require versioning. Deprecated endpoints return a `Deprecation` header with a sunset date.

### 5.2 Complete Route Catalog

#### Authentication — `/api/v1/auth`

| Method | Route | Auth | Description |
|---|---|---|---|
| POST | `/auth/register` | Public | Register with email + password |
| POST | `/auth/login` | Public | Login with email + password |
| POST | `/auth/google` | Public | Initiate Google OAuth |
| GET | `/auth/google/callback` | Public | Google OAuth callback handler |
| POST | `/auth/otp/send` | Public | Send OTP to phone/email |
| POST | `/auth/otp/verify` | Public | Verify OTP code |
| POST | `/auth/refresh` | Public | Refresh access token |
| POST | `/auth/logout` | JWT | Revoke refresh token |
| POST | `/auth/logout/all` | JWT | Revoke all sessions |
| POST | `/auth/forgot-password` | Public | Send password reset email |
| POST | `/auth/reset-password` | Public | Reset with token |
| GET | `/auth/me` | JWT | Get current user profile |

#### Users — `/api/v1/users`

| Method | Route | Auth | Description |
|---|---|---|---|
| GET | `/users/me` | JWT | Full profile |
| PATCH | `/users/me` | JWT | Update profile |
| DELETE | `/users/me` | JWT | Soft-delete account (GDPR) |
| GET | `/users/me/sessions` | JWT | List active sessions |
| DELETE | `/users/me/sessions/:id` | JWT | Revoke specific session |
| PATCH | `/users/me/notifications` | JWT | Update notification preferences |
| POST | `/users/me/fcm-token` | JWT | Register/update FCM device token |
| DELETE | `/users/me/fcm-token` | JWT | Unregister device |

#### Vehicles — `/api/v1/vehicles`

| Method | Route | Auth | Description |
|---|---|---|---|
| GET | `/vehicles` | JWT | List owner's vehicles |
| POST | `/vehicles` | JWT | Add a vehicle |
| GET | `/vehicles/:id` | JWT | Get vehicle detail |
| PATCH | `/vehicles/:id` | JWT | Update vehicle |
| DELETE | `/vehicles/:id` | JWT | Soft-delete vehicle |
| GET | `/vehicles/:id/reports` | JWT | Reports for a vehicle |
| GET | `/vehicles/:id/qr-codes` | JWT | QR codes for a vehicle |

#### QR Codes — `/api/v1/qr`

| Method | Route | Auth | Description |
|---|---|---|---|
| POST | `/qr/generate` | JWT | Generate new QR for a vehicle |
| GET | `/qr/:token/validate` | Public | Validate a QR scan (no owner data returned) |
| GET | `/qr/:id/download` | JWT | Download QR image (PDF/PNG) |
| PATCH | `/qr/:id/deactivate` | JWT | Deactivate a QR code |
| PATCH | `/qr/:id/replace` | JWT | Replace (deactivate old, generate new) |

#### Reports — `/api/v1/reports`

| Method | Route | Auth | Description |
|---|---|---|---|
| POST | `/reports` | Public + CAPTCHA | Submit anonymous report |
| GET | `/reports` | JWT | Owner: list their vehicle reports |
| GET | `/reports/:id` | JWT | Get specific report (owner only) |
| PATCH | `/reports/:id/acknowledge` | JWT | Owner acknowledges report |
| PATCH | `/reports/:id/resolve` | JWT | Owner resolves report |

#### Notifications — `/api/v1/notifications`

| Method | Route | Auth | Description |
|---|---|---|---|
| GET | `/notifications` | JWT | List user notifications |
| PATCH | `/notifications/:id/read` | JWT | Mark single as read |
| PATCH | `/notifications/read-all` | JWT | Mark all as read |
| DELETE | `/notifications/:id` | JWT | Delete notification |

#### Subscriptions — `/api/v1/subscriptions`

| Method | Route | Auth | Description |
|---|---|---|---|
| GET | `/subscriptions/plans` | Public | List available plans |
| GET | `/subscriptions/me` | JWT | Current subscription |
| POST | `/subscriptions/checkout` | JWT | Create payment checkout session |
| POST | `/subscriptions/cancel` | JWT | Cancel subscription |
| POST | `/subscriptions/webhook` | Public (signed) | Payment provider webhook |

#### Storage — `/api/v1/storage`

| Method | Route | Auth | Description |
|---|---|---|---|
| POST | `/storage/signed-url` | JWT | Get Cloudinary signed upload URL |

#### Admin — `/api/v1/admin`

| Method | Route | Auth | Description |
|---|---|---|---|
| GET | `/admin/stats` | ADMIN | Platform-wide metrics |
| GET | `/admin/users` | ADMIN | Paginated user list |
| GET | `/admin/reports` | ADMIN | Moderation queue |
| PATCH | `/admin/reports/:id/flag` | ADMIN | Flag a report |
| DELETE | `/admin/reports/:id` | ADMIN | Remove a report |

### 5.3 Standard API Response Format

All responses follow a consistent envelope:

```
Success (2xx):
{
  "status": "success",
  "data": { ... },
  "meta": {
    "timestamp": "ISO8601",
    "requestId": "uuid",
    "pagination": {              // only on list endpoints
      "page": 1,
      "limit": 20,
      "total": 142,
      "totalPages": 8
    }
  }
}

Error (4xx / 5xx):
{
  "status": "error",
  "error": {
    "code": "VEHICLE_NOT_FOUND",    // machine-readable
    "message": "Human-readable message",
    "details": [...],               // validation errors
    "requestId": "uuid"
  }
}
```

### 5.4 Controller / Service / Repository Pattern

```
HTTP Request
    ↓
Controller  — Receives HTTP, validates with DTOs, delegates to Service
    ↓
Service     — Business logic, orchestrates repositories, external services
    ↓
Repository  — Prisma queries, single table/entity responsibility
    ↓
Database
```

**Why this layering?** Controllers are thin. Services contain all business rules. Repositories are swappable (future: GraphQL resolvers can call the same services). Testing is clean: unit test services with mocked repositories.

---

## 6. Security Architecture

### 6.1 JWT Strategy

| Token | Expiry | Storage | Purpose |
|---|---|---|---|
| Access Token | 15 minutes | Zustand (in-memory) | API authentication |
| Refresh Token | 7 days (rotated) | httpOnly Secure Cookie | Issue new access tokens |

**Why in-memory for access token?** Prevents XSS from stealing tokens via `localStorage`. 15-minute expiry limits blast radius of token theft.

**Refresh token rotation:** Every refresh issues a new refresh token and invalidates the previous one. Parallel requests use a mutex (Redis-based) to prevent token race conditions.

**Token structure (access):**
```
{
  sub: "userId",
  email: "user@example.com",
  roles: ["OWNER"],
  sessionId: "sessionId",
  iat: 1234567890,
  exp: 1234568790
}
```

### 6.2 Session Management

- Sessions stored in `sessions` table (not solely Redis)
- Redis used as fast-path cache for active sessions
- `last_used_at` updated on every refresh
- Admin can force-revoke all sessions for a user
- Suspicious IP changes trigger session invalidation

### 6.3 RBAC Model

```
SUPER_ADMIN  > ADMIN  > OWNER  > (anonymous)

Resources × Actions:
  vehicles    : [create, read, update, delete]
  qr_codes    : [generate, read, deactivate, replace, download]
  reports     : [read, acknowledge, resolve]
  users       : [read:own, update:own, delete:own, read:all (admin)]
  subscriptions: [read:own, manage:own]
  admin.*     : [read:all, update:all, delete:all]
```

RBAC is enforced at two levels:
1. `RolesGuard` at controller method level via `@Roles()` decorator
2. Service-level ownership check: `vehicle.ownerId === request.user.id`

### 6.4 Rate Limiting Strategy

| Endpoint Category | Limit | Window | Scope |
|---|---|---|---|
| Auth (login/register) | 5 req | 1 min | Per IP |
| OTP send | 3 req | 10 min | Per phone |
| Report submission | 10 req | 1 hour | Per IP |
| QR scan/validate | 100 req | 1 min | Per IP |
| Authenticated API | 200 req | 1 min | Per User |
| Admin API | 500 req | 1 min | Per Admin |

Implemented via NestJS Throttler with Redis storage. NGINX handles burst protection at the edge.

### 6.5 CAPTCHA

Report submission (`POST /reports`) requires a valid CAPTCHA token (hCaptcha or Cloudflare Turnstile). Verified server-side before processing. Prevents automated report flooding.

### 6.6 Input Validation & PII Scrubbing

- All DTOs validated via `class-validator` with whitelist mode (unknown properties stripped)
- Report `description` field runs through a PII scrubber (regex + optional ML) to remove detected emails, phone numbers, license plate patterns, and names before storage
- XSS prevention: HTML-entity encoding on all string outputs

### 6.7 Encryption & Hashing

| Data | Method |
|---|---|
| Passwords | bcrypt, cost factor 12 |
| OTP codes | SHA-256 hash stored, not plaintext |
| Refresh tokens | SHA-256 hash stored in DB (not raw token) |
| Vehicle owner email/phone in notifications | AES-256-GCM at rest (env key) |
| Database at rest | AWS RDS encryption (AES-256) |
| Data in transit | TLS 1.3 everywhere |

### 6.8 Security Headers (NGINX + Next.js)

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Content-Security-Policy: [strict policy]
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=(self)
```

### 6.9 Audit Logging

Every state-changing API action logs to `audit_logs`:

- Who (userId or "anonymous")
- What action
- On what resource (type + id)
- Before and after state (jsonb)
- IP address, user agent
- Correlation ID (traceable through distributed logs)

Audit logs are immutable (no UPDATE or DELETE permissions at DB level for audit table).

### 6.10 Privacy Architecture

```
Privacy Boundary
─────────────────────────────────────────
       LEFT                RIGHT
  (Scanner World)    (Owner World)
  
  anonymous_session_id  ←── NO LINK ──→  user_id
  report content              ↑          vehicle info
  photo URLs           Platform           contact info
  location coords      mediates          notification prefs
─────────────────────────────────────────
```

- No scanner user account required
- Anonymous session ID generated client-side (not stored beyond correlation)
- Report response to scanner contains zero owner data
- Owner notification contains zero scanner data
- Platform support cannot join scanner identity to report without extraordinary effort (by design)

---

## 7. Feature Roadmap

### MVP (Month 1–2): Core Loop Working

**Goal:** One user can register, add a vehicle, get a QR code, and receive a notification when the QR is scanned.

| Feature | Priority | Effort |
|---|---|---|
| User registration + email/password auth | P0 | M |
| Add vehicle (make, model, plate, color) | P0 | S |
| Generate QR code for vehicle | P0 | S |
| QR scan → anonymous report form (web) | P0 | M |
| Report types: Wrong Parking, Lights On, Emergency | P0 | S |
| Email notification to owner on report | P0 | S |
| Owner dashboard: view reports | P0 | M |
| Basic subscription: Free tier (1 vehicle, 1 QR) | P0 | M |

**Out of scope for MVP:** SMS, push notifications, photos, maps, Google OAuth

---

### Version 1.0 (Month 3–4): Production-Ready

**Goal:** Full notification channels, media uploads, maps, proper auth.

| Feature | Priority | Effort |
|---|---|---|
| Google OAuth login | P1 | S |
| OTP phone authentication | P1 | M |
| SMS notifications via Twilio | P1 | M |
| FCM Push Notifications (web + mobile) | P1 | L |
| Photo upload with reports (Cloudinary) | P1 | M |
| Google Maps location tagging on reports | P1 | M |
| Owner acknowledge / resolve reports | P1 | S |
| QR code download (PDF/PNG for printing) | P1 | S |
| Paid subscription tiers (Basic / Pro) | P1 | L |
| Rate limiting + CAPTCHA on reports | P1 | M |
| Full RBAC + Admin dashboard | P1 | L |
| Audit logging | P1 | S |

---

### Version 2.0 (Month 5–7): Growth Features

**Goal:** Increase engagement, add premium value, expand device support.

| Feature | Priority | Effort |
|---|---|---|
| Mobile app (React Native or PWA) | P2 | XL |
| Multi-language support (i18n) | P2 | L |
| Real-time notifications (WebSocket) | P2 | M |
| Report analytics for owners (charts, trends) | P2 | L |
| QR sticker order & fulfillment integration | P2 | L |
| Emergency report → direct call-to-action (owner phone masked via Twilio proxy) | P2 | L |
| Owner sets "away" mode (auto-reply to reporters) | P2 | M |
| Report deduplication (prevent 50 reports for same incident) | P2 | M |
| Zapier / webhook integrations | P2 | M |
| API access for enterprise customers | P2 | L |

---

### Version 3.0 (Month 8–12): Platform Scale

**Goal:** B2B expansion, fleet management, AI-powered features.

| Feature | Priority | Effort |
|---|---|---|
| Fleet management for businesses (100+ vehicles) | P3 | XL |
| AI-powered report summarization | P3 | M |
| AI photo analysis (detect damage, identify issues) | P3 | L |
| White-label solution for parking operators | P3 | XL |
| Native iOS & Android app | P3 | XL |
| Offline QR code scanning (cached validation) | P3 | M |
| Integration with parking management systems | P3 | XL |
| NFC tag support (alongside QR) | P3 | L |
| Municipal/government partnership tier | P3 | XL |

---

## 8. Technical Documentation

### 8.1 Architecture Decisions

#### ADR-001: Turborepo Monorepo

**Decision:** Single Turborepo monorepo containing all apps and shared packages.

**Rationale:**
- Type sharing between frontend and backend eliminates interface drift
- Single PR touches both frontend and API changes atomically
- Turborepo remote caching reduces CI build times by 60-80%
- Shared ESLint/Prettier/TypeScript config enforces consistency

**Trade-offs:** More complex initial setup. Entire repo must be cloned. Mitigated by sparse checkout for deployment.

---

#### ADR-002: NestJS for Backend

**Decision:** NestJS over Express, Fastify, or Hono.

**Rationale:**
- Angular-inspired module system scales to large teams without chaos
- Dependency injection makes unit testing first-class
- Built-in Swagger/OpenAPI generation from decorators
- Guards, interceptors, pipes, filters are perfect for cross-cutting concerns
- Native TypeScript — no bolt-on types

**Trade-offs:** More boilerplate than Express. Steeper learning curve. Slower cold start than Hono (acceptable for a server — not a Lambda).

---

#### ADR-003: Prisma ORM

**Decision:** Prisma over TypeORM, Drizzle, or raw SQL.

**Rationale:**
- Schema-first with auto-generated TypeScript types
- Migration system is predictable and version-controlled
- Prisma Studio for non-engineer DB inspection
- `prisma.$transaction` for atomic multi-table operations
- Excellent NestJS integration

**Trade-offs:** Prisma Client generates large bundles (mitigated by server-side use only). Complex queries require raw SQL fallback.

---

#### ADR-004: Privacy-First Data Model

**Decision:** Reports are never linked to scanner identity at the database level.

**Rationale:** This is the core product promise. Legal simplicity (GDPR/DPDP), trust with both user types, and competitive differentiation. The platform is less valuable if either party fears exposure.

**Trade-offs:** Abuse prevention is harder (can't ban a specific scanner). Mitigated by IP-based rate limiting, CAPTCHA, and report flagging.

---

#### ADR-005: Redis for Sessions, OTP, and Rate Limiting

**Decision:** Redis (AWS ElastiCache) alongside PostgreSQL.

**Rationale:**
- OTPs need 5-min TTL with atomic compare-and-delete — Redis SET EX is perfect
- Rate limit counters need sub-millisecond increments — Redis INCR/EXPIRE
- Session cache avoids DB hit on every API request

**Trade-offs:** Additional infrastructure cost (~$15-30/mo for t3.micro ElastiCache). Sessions also persisted to PostgreSQL as fallback.

---

### 8.2 Technology Decisions

| Technology | Version | Why Chosen | Alternative Considered |
|---|---|---|---|
| Next.js | 15 | App Router, RSC, best-in-class DX | Remix, Vite + React |
| NestJS | 10.x | Scalable structure, DI, TypeScript-native | Express, Fastify |
| PostgreSQL | 16 | ACID, JSONB, full-text search, row-level security | MySQL, MongoDB |
| Prisma | 5.x | Type-safe ORM, excellent migrations | TypeORM, Drizzle |
| Redis | 7.x | Fast ephemeral store, atomic ops | Memcached |
| Cloudinary | — | Image transformation, CDN, signed uploads | AWS S3 + CloudFront |
| Firebase FCM | — | Free push notifications, excellent reliability | OneSignal, AWS SNS |
| Twilio | — | Industry standard SMS/OTP, global coverage | AWS SNS, Vonage |
| TailwindCSS | 4.x | Utility-first, purge = tiny bundles | Styled Components |
| ShadCN UI | — | Accessible, composable, not a locked-in library | Radix UI direct, MUI |
| React Query | 5.x | Server state, caching, background refetch | SWR, Apollo |
| Zustand | 4.x | Minimal, no boilerplate, TypeScript-first | Redux Toolkit, Jotai |
| Turborepo | 2.x | Build caching, parallel tasks, workspace mgmt | Nx, Lerna |
| pnpm | 9.x | Strict, fast, disk-efficient | npm, yarn |

---

### 8.3 Scalability Considerations

#### Database Scaling Path

```
Phase 1 (MVP):        Single RDS t3.medium instance (~500 users)
Phase 2 (Growth):     Primary + Read Replica (~10K users)
Phase 3 (Scale):      Connection pooling via PgBouncer + Read replicas
Phase 4 (Hyperscale): Partitioned tables (reports by created_at month)
                      Consider Aurora Serverless for variable load
```

**Indexes to plan from day one:**
- `qr_codes.token` — scanned millions of times
- `reports.qr_code_id` — owner's report list queries
- `reports.created_at` — time-series queries
- `notifications.user_id + status` — unread count queries
- `vehicles.owner_id` — dashboard queries

#### Application Scaling Path

```
Phase 1: Single NestJS instance (Docker on EC2/ECS)
Phase 2: ECS Fargate: auto-scale API service by CPU/request count
Phase 3: Separate services: notifications as independent worker
Phase 4: Event-driven (SQS/SNS) for notification delivery
```

#### CDN and Media

- QR code images: generated once, stored Cloudinary, cached aggressively (immutable, URL contains hash)
- Report photos: user-uploaded, Cloudinary auto-transforms for web/mobile
- Next.js static assets: Cloudflare CDN with long cache TTL

---

### 8.4 Cost Considerations (Estimated Monthly, at MVP scale ~1,000 active users)

| Service | Tier | Estimated Cost |
|---|---|---|
| AWS EC2 / ECS Fargate (API) | t3.small | ~$20 |
| AWS RDS PostgreSQL | t3.micro, 20GB | ~$25 |
| AWS ElastiCache Redis | t3.micro | ~$15 |
| Cloudinary | Free tier (25GB) | $0 |
| Firebase FCM | Free | $0 |
| Twilio SMS | ~0.0075/SMS, 1000 SMS/mo | ~$8 |
| SendGrid Email | Free (100/day) | $0 |
| Google Maps API | $0.002/request, 10K req/mo | ~$20 |
| Cloudflare (CDN + DNS) | Free tier | $0 |
| Domain + SSL | — | ~$15/yr |
| **Total** | | **~$90/month** |

**Break-even:** At $5/mo per paid subscriber, break-even is ~18 paid users. Easily achievable.

**Cost at 10,000 users (~10x scale):**
- RDS: upgrade to t3.medium + read replica → ~$90
- Fargate: scale to 2 tasks → ~$60
- Twilio: ~$80
- Google Maps: ~$200
- **Total: ~$500/month** → requires ~100 paid subscribers at $5/mo

---

### 8.5 Monitoring & Observability Strategy

```
Layer 1 — Infrastructure: AWS CloudWatch for ECS, RDS, ElastiCache metrics
Layer 2 — Application: Pino structured JSON logs → CloudWatch Logs / Loki
Layer 3 — APM: OpenTelemetry → Grafana Tempo (traces)
Layer 4 — Error Tracking: Sentry (both Next.js and NestJS)
Layer 5 — Uptime: UptimeRobot or Better Stack (public status page)
Layer 6 — Business Metrics: Custom dashboard (reports/day, QR scans, active users)
```

**Alerting Rules:**
- API error rate > 5% → PagerDuty
- Response P99 > 2s → Slack warning
- DB connection pool exhausted → PagerDuty
- Notification delivery failure rate > 10% → Slack warning
- Failed payment webhooks → Email to billing team

---

### 8.6 Deployment Strategy

```
Branch          Environment     Deploy Trigger
──────────────────────────────────────────────────
feature/*       PR Preview      Manual / PR open
develop         Staging         Merge to develop
main            Production      Git tag (v1.x.x)
```

**Zero-downtime deployments:**
1. ECS rolling deployment (50% at a time)
2. Prisma migrations run **before** code deployment (backward-compatible migrations always)
3. Health check endpoint (`/api/health`) gates deployment progression

**Rollback procedure:**
1. ECS: point task definition back to previous image tag
2. Database: migration down scripts in `/prisma/migrations`
3. Documented in `/docs/runbooks/rollback.md`

---

### 8.7 Environment Variables Reference

```bash
# Application
NODE_ENV=production
PORT=3001
APP_URL=https://smartvehicleqr.com
API_URL=https://api.smartvehicleqr.com

# Database
DATABASE_URL=postgresql://user:pass@host:5432/svqr_prod
DATABASE_READ_URL=postgresql://user:pass@read-replica:5432/svqr_prod

# Redis
REDIS_URL=redis://host:6379

# JWT
JWT_ACCESS_SECRET=<256-bit-random>
JWT_REFRESH_SECRET=<256-bit-random>
JWT_ACCESS_EXPIRY=15m
JWT_REFRESH_EXPIRY=7d

# Google OAuth
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_CALLBACK_URL=

# Twilio
TWILIO_ACCOUNT_SID=
TWILIO_AUTH_TOKEN=
TWILIO_PHONE_NUMBER=

# Firebase
FIREBASE_PROJECT_ID=
FIREBASE_PRIVATE_KEY=
FIREBASE_CLIENT_EMAIL=

# Cloudinary
CLOUDINARY_CLOUD_NAME=
CLOUDINARY_API_KEY=
CLOUDINARY_API_SECRET=

# Email
SENDGRID_API_KEY=
EMAIL_FROM=noreply@smartvehicleqr.com

# Google Maps
GOOGLE_MAPS_API_KEY=

# CAPTCHA
CAPTCHA_SECRET_KEY=

# Encryption
ENCRYPTION_KEY=<256-bit-random-aes-key>

# Payments
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
```

---

## Appendix A: Naming Conventions

| Layer | Convention | Example |
|---|---|---|
| Files | kebab-case | `vehicles.service.ts` |
| Classes | PascalCase | `VehiclesService` |
| Variables/Functions | camelCase | `getVehicleById` |
| Database tables | snake_case | `qr_codes` |
| Database columns | snake_case | `owner_id`, `created_at` |
| API routes | kebab-case | `/api/v1/qr-codes` |
| Env variables | SCREAMING_SNAKE | `JWT_ACCESS_SECRET` |
| React components | PascalCase | `VehicleCard.tsx` |
| Zustand stores | camelCase + Store | `authStore.ts` |
| React Query keys | kebab-case arrays | `['vehicles', vehicleId]` |

---

## Appendix B: Report Types Reference

| Code | Display Name | Severity | Description |
|---|---|---|---|
| `WRONG_PARKING` | Wrong Parking | LOW | Vehicle parked incorrectly |
| `BLOCKED_VEHICLE` | Blocking Vehicle | MEDIUM | Vehicle blocking exit/entrance |
| `LIGHTS_ON` | Headlights Left On | LOW | Lights left on, battery drain risk |
| `ENGINE_RUNNING` | Engine Running | MEDIUM | Unattended engine running |
| `ACCIDENT` | Accident | HIGH | Vehicle involved in collision |
| `EMERGENCY` | Emergency | CRITICAL | Immediate attention required |
| `OTHER` | Other | LOW | Custom free-text report |

---

*Document maintained by: Engineering Team*  
*Last updated: Architecture v1.0*  
*Next review: Post-MVP launch*
