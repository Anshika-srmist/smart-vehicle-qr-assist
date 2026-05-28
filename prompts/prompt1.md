You are a senior software architect and CTO.

Create the complete architecture and project foundation for a production-ready SaaS application called:

Smart Vehicle QR Assist

Purpose:
A platform where vehicle owners place a QR code on their vehicle. Anyone encountering an issue (wrong parking, blocked vehicle, lights left ON, accident, emergency) can scan the QR and anonymously notify the owner.

The owner never sees the scanner's personal information.

The scanner never sees the owner's personal information.

━━━━━━━━━━━━━━━━━━━━━━

TECH STACK

Frontend:
- Next.js 15
- TypeScript
- TailwindCSS
- ShadCN UI
- React Query
- Zustand

Backend:
- NestJS
- TypeScript
- Prisma ORM

Database:
- PostgreSQL

Storage:
- Cloudinary

Notifications:
- Firebase Cloud Messaging
- Email
- SMS
- Twilio

Maps:
- Google Maps API

Authentication:
- JWT
- Google OAuth
- OTP

━━━━━━━━━━━━━━━━━━━━━━

I need the following:

1. Complete Monorepo Structure

Create the entire repository folder architecture.

Include:

apps/
packages/
services/
docs/
tests/
.github/

Show every folder and file.

Explain why each exists.

━━━━━━━━━━━━━━━━━━━━━━

2. Software Architecture

Generate:

- System Architecture Diagram
- Component Diagram
- Request Flow Diagram
- Authentication Flow
- Notification Flow
- QR Scan Flow
- Report Submission Flow

Use Mermaid diagrams.

━━━━━━━━━━━━━━━━━━━━━━

3. Database Planning

Design all entities.

Include:

Users
Vehicles
QR Codes
Reports
Notifications
Subscriptions
Audit Logs
Sessions
Roles
Permissions

Show:

- Entity Relationship Diagram
- Table Relationships
- Data Flow

━━━━━━━━━━━━━━━━━━━━━━

4. API Architecture

Design complete REST architecture.

Include:

Versioning strategy

/api/v1/

Folder structure

Controllers

Services

Repositories

DTOs

Validators

Middlewares

Guards

Interceptors

Filters

Provide route naming conventions.

━━━━━━━━━━━━━━━━━━━━━━

5. Security Architecture

Design:

JWT Strategy
Refresh Tokens
Session Management
RBAC
Rate Limiting
CAPTCHA
Input Validation
Password Hashing
Encryption
Audit Logging

━━━━━━━━━━━━━━━━━━━━━━

6. Feature Roadmap

Create:

MVP
Version 1
Version 2
Version 3

Prioritize features.

━━━━━━━━━━━━━━━━━━━━━━

7. Generate technical documentation.

Include:

Architecture decisions
Technology decisions
Scalability considerations
Cost considerations

Output everything in markdown.

Do not generate code yet.

Focus only on architecture and planning.