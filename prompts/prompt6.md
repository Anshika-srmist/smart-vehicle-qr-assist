# Prompt 6 — Notification System + AI Voice Calls

Continue Smart Vehicle QR Assist.

Focus ONLY on:

1. Notification System
2. Push Notifications
3. SMS Notifications
4. Email Notifications
5. AI Voice Calls
6. Escalation System

━━━━━━━━━━━━━━━━━━━━━━

# SERVICES

Integrate:

* Firebase Cloud Messaging
* Twilio
* Nodemailer

━━━━━━━━━━━━━━━━━━━━━━

# NOTIFICATION TYPES

Support:

* Push Notification
* SMS
* Email
* Voice Call
* In-App Notification

━━━━━━━━━━━━━━━━━━━━━━

# REPORT ALERT FLOW

User submits report
↓
System creates notification
↓
Push notification sent
↓
SMS sent
↓
Voice call initiated
↓
Owner acknowledges
↓
Notifications stop

━━━━━━━━━━━━━━━━━━━━━━

# AI CALL FLOW

Integrate Twilio Voice API.

Features:

* text-to-speech
* issue announcement
* retry logic
* acknowledgement detection

━━━━━━━━━━━━━━━━━━━━━━

# RETRY LOGIC

If owner ignores call:

Retry:

* after 1 minute
* after 5 minutes
* after 15 minutes
* after 30 minutes

Stop after acknowledgement.

━━━━━━━━━━━━━━━━━━━━━━

# BACKEND REQUIREMENTS

Generate:

notifications/
├── notifications.controller.ts
├── notifications.service.ts
├── notifications.module.ts
├── providers/
├── templates/

━━━━━━━━━━━━━━━━━━━━━━

# DATABASE MODEL

Notification fields:

* id
* userId
* reportId
* type
* title
* message
* status
* sentAt
* readAt

━━━━━━━━━━━━━━━━━━━━━━

# FRONTEND REQUIREMENTS

Generate:

* Notification Center
* Notification Dropdown
* Notification Preferences
* Alert History

━━━━━━━━━━━━━━━━━━━━━━

# USER SETTINGS

Allow users to:

* enable/disable SMS
* enable/disable calls
* mute notifications
* configure emergency mode

━━━━━━━━━━━━━━━━━━━━━━

# UI COMPONENTS

Generate:

* NotificationBell
* NotificationPanel
* AlertToast
* NotificationCard
* CallStatusIndicator

━━━━━━━━━━━━━━━━━━━━━━

# REAL-TIME SYSTEM

Implement:

* websocket updates
* live notifications
* unread counts

━━━━━━━━━━━━━━━━━━━━━━

# EMAIL TEMPLATES

Create:

* report alert email
* emergency alert email
* verification email

━━━━━━━━━━━━━━━━━━━━━━

# SECURITY

Prevent:

* notification spam
* abuse
* excessive retries

━━━━━━━━━━━━━━━━━━━━━━

# TESTING

Generate:

* notification tests
* Twilio tests
* websocket tests

━━━━━━━━━━━━━━━━━━━━━━

# IMPORTANT

Generate exact file paths.

Use production-ready architecture only.
