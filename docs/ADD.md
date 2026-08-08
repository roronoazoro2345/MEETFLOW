================================================================================
                      API DESIGN DOCUMENT (ADD)
================================================================================
Project Name: MeetFlow
Architectural Style: RESTful Web API
Technology Stack: ASP.NET Core Web API (C#)
Document Version: 1.0

--------------------------------------------------------------------------------
1. GLOBAL API STANDARDS
--------------------------------------------------------------------------------
Base URL (Dev): https://localhost:7001/api/v1
Base URL (Prod): https://api.meetflow.com/api/v1

Headers:
- Authorization: Bearer <JWT_TOKEN>
- Content-Type: application/json
- Accept: application/json

Standard JSON Success Envelope:
{
  "success": true,
  "message": "Operation completed successfully.",
  "data": { ... },
  "errors": null,
  "timestamp": "2026-08-08T12:00:00Z"
}

Standard JSON Error Envelope:
{
  "success": false,
  "message": "Validation failed.",
  "data": null,
  "errors": [ "Error details here" ],
  "timestamp": "2026-08-08T12:00:00Z"
}

--------------------------------------------------------------------------------
2. ENDPOINTS SPECIFICATION
--------------------------------------------------------------------------------

2.1 AUTHENTICATION ROUTE (/auth)
- POST /auth/login
  Access: Public
  Body: { "email": "user@meetflow.com", "password": "Password123!" }
  Returns: JWT token, expiration, user claims.

- POST /auth/forgot-password
  Access: Public
  Body: { "email": "user@meetflow.com" }

- POST /auth/reset-password
  Access: Public
  Body: { "token": "...", "newPassword": "..." }

2.2 DASHBOARD ROUTE (/dashboard)
- GET /dashboard/manager
  Access: Manager
  Returns: Meetings today, pending tasks, team performance score.

- GET /dashboard/employee
  Access: Employee
  Returns: Assigned tasks, due tasks, today's schedule.

2.3 MEETING MANAGEMENT ROUTE (/meetings)
- GET /meetings
  Access: Admin, Manager, Employee
  Params: page, pageSize, status, startDate, endDate

- POST /meetings
  Access: Manager
  Body: { "title": "...", "agenda": "...", "meetingType": "Online", "provider": "Google Meet", "startTime": "...", "endTime": "...", "participantUserIds": [1, 2] }
  Triggers: Auto-generation of meeting link via Google/MS API.

- PUT /meetings/{id}/reschedule
  Access: Manager
  Body: { "newStartTime": "...", "newEndTime": "..." }

2.4 MINUTES OF MEETING ROUTE (/meetings/{meetingId}/mom)
- POST /meetings/{meetingId}/mom
  Access: Manager
  Body: { "summary": "...", "discussionPoints": "...", "decisionsTaken": "...", "risks": "...", "nextSteps": "..." }

- GET /meetings/{meetingId}/mom
  Access: All Meeting Attendees

2.5 TASKS ROUTE (/tasks)
- POST /tasks
  Access: Manager
  Body: { "meetingId": 10, "assignedToUserId": 4, "title": "...", "dueDate": "...", "priority": "High" }

- PATCH /tasks/{id}/status
  Access: Assigned Employee, Manager
  Body: { "status": "In Progress", "remarks": "..." }

- POST /tasks/{id}/comments
  Access: Assigned Employee, Manager
  Body: { "commentText": "..." }

2.6 REPORTS ROUTE (/reports)
- GET /reports/meeting-effectiveness
  Access: Admin, Manager
  Params: departmentId, startDate, endDate

- GET /reports/export
  Access: Admin, Manager
  Params: reportType, format (PDF/Excel)
  Returns: File Stream Response

--------------------------------------------------------------------------------
3. REAL-TIME SIGNALR HUBS (/hubs/notifications)
--------------------------------------------------------------------------------
Server-to-Client Events:
- ReceiveNotification: Pushes alerts for scheduled/updated meetings.
- TaskAssigned: Notifies employee when a task is created.
- TaskStatusChanged: Notifies manager when employee updates task status.
