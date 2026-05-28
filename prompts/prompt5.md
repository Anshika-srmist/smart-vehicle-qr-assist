# Prompt 5 — Anonymous Reporting System

Continue Smart Vehicle QR Assist.

Focus ONLY on:

1. Anonymous Reporting
2. Public QR Interaction
3. Report Management
4. Emergency Reporting

━━━━━━━━━━━━━━━━━━━━━━

# CORE CONCEPT

A user scans a QR code on a vehicle and anonymously reports an issue.

The scanner never sees owner information.

The owner never sees scanner information.

━━━━━━━━━━━━━━━━━━━━━━

# REPORT TYPES

Support:

* Blocking Vehicle
* Blocking Traffic
* Lights Left ON
* Window Open
* Minor Accident
* Vehicle Damage
* Suspicious Activity
* Emergency
* Custom Message

━━━━━━━━━━━━━━━━━━━━━━

# REPORT FLOW

Scan QR
↓
Open public page
↓
Select issue
↓
Add optional message
↓
Add location
↓
Add photos
↓
Submit anonymously
↓
Owner receives notification

━━━━━━━━━━━━━━━━━━━━━━

# BACKEND REQUIREMENTS

Generate:

reports/
├── reports.controller.ts
├── reports.service.ts
├── reports.module.ts
├── reports.repository.ts
├── dto/
├── entities/

━━━━━━━━━━━━━━━━━━━━━━

# REPORT MODEL

Fields:

* id
* qrCodeId
* vehicleId
* type
* description
* status
* latitude
* longitude
* photos
* priority
* createdAt
* resolvedAt

━━━━━━━━━━━━━━━━━━━━━━

# REPORT STATUS

Support:

* PENDING
* DELIVERED
* ACKNOWLEDGED
* RESOLVED
* IGNORED

━━━━━━━━━━━━━━━━━━━━━━

# API ENDPOINTS

POST /reports
GET /reports
GET /reports/:id
PATCH /reports/:id/status

━━━━━━━━━━━━━━━━━━━━━━

# EMERGENCY REPORTING

Emergency reports should:

* trigger high priority
* bypass standard retry delays
* trigger immediate notifications

━━━━━━━━━━━━━━━━━━━━━━

# SPAM PROTECTION

Implement:

* CAPTCHA
* rate limiting
* IP throttling
* abuse detection

━━━━━━━━━━━━━━━━━━━━━━

# IMAGE UPLOADS

Allow:

* multiple images
* compression
* Cloudinary upload

━━━━━━━━━━━━━━━━━━━━━━

# FRONTEND REQUIREMENTS

Generate:

reports/
├── page.tsx
├── [id]/
├── create/

━━━━━━━━━━━━━━━━━━━━━━

# PUBLIC REPORT PAGE

Public route:

/qr/[token]

Features:

* issue selection
* anonymous form
* location picker
* image upload
* emergency mode
* success confirmation

━━━━━━━━━━━━━━━━━━━━━━

# OWNER DASHBOARD

Vehicle owner can:

* view reports
* resolve reports
* acknowledge reports
* filter reports
* search reports

━━━━━━━━━━━━━━━━━━━━━━

# UI COMPONENTS

Generate:

* ReportCard
* ReportTimeline
* ReportForm
* EmergencyBanner
* StatusBadge
* ImageUploader

━━━━━━━━━━━━━━━━━━━━━━

# REAL-TIME FEATURES

Implement:

* live status updates
* optimistic updates
* notification refresh

━━━━━━━━━━━━━━━━━━━━━━

# TESTING

Generate:

* API tests
* frontend tests
* validation tests

━━━━━━━━━━━━━━━━━━━━━━

# IMPORTANT

Generate production-ready code only.

Use modular architecture.

Provide exact file paths.
