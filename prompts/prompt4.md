# Prompt 4 — QR System + Vehicle Management

Continue building Smart Vehicle QR Assist.

The backend foundation and frontend foundation already exist.

Do NOT regenerate existing architecture or infrastructure.

Focus ONLY on:

1. Vehicle Management System
2. QR Code System

━━━━━━━━━━━━━━━━━━━━━━

# STACK

Frontend:

* Next.js 15
* TypeScript
* TailwindCSS
* ShadCN UI
* Zustand
* React Query

Backend:

* NestJS
* Prisma
* PostgreSQL

QR:

* qrcode npm package
* dynamic QR generation

━━━━━━━━━━━━━━━━━━━━━━

# VEHICLE MANAGEMENT REQUIREMENTS

Implement complete CRUD operations for vehicles.

Each authenticated user can:

* Add vehicle
* Edit vehicle
* Delete vehicle
* View vehicles
* Search vehicles
* Filter vehicles
* Upload vehicle image
* Generate QR
* Download QR
* Temporarily disable QR
* Regenerate QR

━━━━━━━━━━━━━━━━━━━━━━

# VEHICLE FIELDS

Vehicle should include:

* id
* userId
* vehicleNumber
* vehicleType
* brand
* model
* year
* color
* nickname
* profileImage
* qrCodeId
* createdAt
* updatedAt

━━━━━━━━━━━━━━━━━━━━━━

# VEHICLE TYPES

Support:

* Car
* Bike
* Scooter
* Truck
* Bus
* EV

━━━━━━━━━━━━━━━━━━━━━━

# BACKEND REQUIREMENTS

Generate:

vehicles/
├── vehicles.controller.ts
├── vehicles.service.ts
├── vehicles.module.ts
├── vehicles.repository.ts
├── dto/
├── entities/

━━━━━━━━━━━━━━━━━━━━━━

# API ENDPOINTS

Create:

POST /vehicles
GET /vehicles
GET /vehicles/:id
PATCH /vehicles/:id
DELETE /vehicles/:id

━━━━━━━━━━━━━━━━━━━━━━

# VALIDATION

Validate:

* unique vehicle number
* valid year
* required fields
* ownership verification

━━━━━━━━━━━━━━━━━━━━━━

# SECURITY

Ensure:

* only owner can modify vehicle
* admins can moderate
* JWT protected routes

━━━━━━━━━━━━━━━━━━━━━━

# QR SYSTEM REQUIREMENTS

Each vehicle must generate:

* unique QR code
* public anonymous URL

Example:

https://yourdomain.com/qr/abc123

━━━━━━━━━━━━━━━━━━━━━━

# QR FEATURES

Generate:

* PNG QR
* SVG QR
* print-ready version
* downloadable QR
* sticker preview

━━━━━━━━━━━━━━━━━━━━━━

# QR DATABASE MODEL

Include:

* qrId
* vehicleId
* token
* isActive
* createdAt
* lastScannedAt
* scanCount

━━━━━━━━━━━━━━━━━━━━━━

# QR FLOW

User registers vehicle
↓
System generates QR
↓
QR stored in database
↓
Public scan page created
↓
Anonymous reporting enabled

━━━━━━━━━━━━━━━━━━━━━━

# FRONTEND REQUIREMENTS

Generate:

vehicles/
├── page.tsx
├── create/
├── [id]/
├── edit/

Create components:

* VehicleCard
* VehicleForm
* VehicleTable
* VehicleDetails
* QRDisplay
* QRDownloadModal
* QRPreview

━━━━━━━━━━━━━━━━━━━━━━

# UI REQUIREMENTS

Support:

* dark mode
* light mode
* mobile responsive
* loading states
* skeletons
* empty states
* animations

━━━━━━━━━━━━━━━━━━━━━━

# QR PUBLIC PAGE

Create public route:

/qr/[token]

Features:

* no authentication required
* vehicle info limited for privacy
* issue selection form
* anonymous report form

━━━━━━━━━━━━━━━━━━━━━━

# IMAGE STORAGE

Integrate Cloudinary for:

* vehicle images
* QR storage

━━━━━━━━━━━━━━━━━━━━━━

# TESTING

Generate:

* backend unit tests
* frontend component tests
* API tests

━━━━━━━━━━━━━━━━━━━━━━

# IMPORTANT

Generate:

* exact file paths
* modular structure
* production-ready code
* reusable components

Do not regenerate already existing infrastructure.
