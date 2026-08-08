================================================================================
                    DATABASE DESIGN DOCUMENT (DDD)
================================================================================
Project Name: MeetFlow
Database Engine: Microsoft SQL Server
ORM: Entity Framework Core
Document Version: 1.0

--------------------------------------------------------------------------------
1. DATA DICTIONARY & TABLE DEFINITIONS
--------------------------------------------------------------------------------

1.1 Departments
Stores organizational department structure.
- DepartmentId (INT, Primary Key, IDENTITY)
- DepartmentName (NVARCHAR(100), NOT NULL)
- CreatedAt (DATETIME2, NOT NULL, Default: GETUTCDATE())

1.2 Roles
Defines system access permissions.
- RoleId (INT, Primary Key, IDENTITY)
- RoleName (NVARCHAR(50), NOT NULL, UNIQUE) -- Admin, Manager, Employee

1.3 Users
Holds user profile and authentication credentials.
- UserId (INT, Primary Key, IDENTITY)
- DepartmentId (INT, Foreign Key -> Departments.DepartmentId, NULLABLE)
- RoleId (INT, Foreign Key -> Roles.RoleId, NOT NULL)
- FullName (NVARCHAR(100), NOT NULL)
- Email (NVARCHAR(150), NOT NULL, UNIQUE)
- PasswordHash (NVARCHAR(MAX), NOT NULL)
- IsActive (BIT, NOT NULL, Default: 1)
- CreatedAt (DATETIME2, NOT NULL, Default: GETUTCDATE())

1.4 Meetings
Contains meeting metadata and video conferencing details.
- MeetingId (INT, Primary Key, IDENTITY)
- OrganizerId (INT, Foreign Key -> Users.UserId, NOT NULL)
- Title (NVARCHAR(200), NOT NULL)
- Agenda (NVARCHAR(MAX), NULLABLE)
- MeetingType (NVARCHAR(20), NOT NULL) -- Online / Offline
- Provider (NVARCHAR(50), NULLABLE) -- Google Meet / Microsoft Teams
- MeetingLink (NVARCHAR(MAX), NULLABLE)
- Priority (NVARCHAR(20), NOT NULL, Default: 'Medium') -- Low, Medium, High
- StartTime (DATETIME2, NOT NULL)
- EndTime (DATETIME2, NOT NULL)
- Status (NVARCHAR(20), NOT NULL, Default: 'Scheduled') -- Scheduled, Completed, Cancelled
- CreatedAt (DATETIME2, NOT NULL, Default: GETUTCDATE())

1.5 MeetingParticipants
Junction table tracking meeting attendees.
- MeetingParticipantId (INT, Primary Key, IDENTITY)
- MeetingId (INT, Foreign Key -> Meetings.MeetingId, NOT NULL)
- UserId (INT, Foreign Key -> Users.UserId, NOT NULL)
- IsAttended (BIT, NOT NULL, Default: 0)

1.6 MeetingMoMs
Records notes, decisions, and summaries following a meeting.
- MoMId (INT, Primary Key, IDENTITY)
- MeetingId (INT, Foreign Key -> Meetings.MeetingId, NOT NULL, UNIQUE)
- CreatedBy (INT, Foreign Key -> Users.UserId, NOT NULL)
- Summary (NVARCHAR(MAX), NOT NULL)
- DiscussionPoints (NVARCHAR(MAX), NULLABLE)
- DecisionsTaken (NVARCHAR(MAX), NULLABLE)
- Risks (NVARCHAR(MAX), NULLABLE)
- NextSteps (NVARCHAR(MAX), NULLABLE)
- FollowUpMeetingDate (DATETIME2, NULLABLE)
- CreatedAt (DATETIME2, NOT NULL, Default: GETUTCDATE())

1.7 Tasks
Tracks action items created from meetings.
- TaskId (INT, Primary Key, IDENTITY)
- MeetingId (INT, Foreign Key -> Meetings.MeetingId, NOT NULL)
- AssignedToUserId (INT, Foreign Key -> Users.UserId, NOT NULL)
- CreatedByUserId (INT, Foreign Key -> Users.UserId, NOT NULL)
- Title (NVARCHAR(200), NOT NULL)
- Description (NVARCHAR(MAX), NULLABLE)
- Priority (NVARCHAR(20), NOT NULL, Default: 'Medium')
- EstimatedHours (DECIMAL(5,2), NULLABLE)
- DueDate (DATETIME2, NOT NULL)
- Status (NVARCHAR(20), NOT NULL, Default: 'Pending') -- Pending, In Progress, Blocked, Completed
- Remarks (NVARCHAR(MAX), NULLABLE)
- CreatedAt (DATETIME2, NOT NULL, Default: GETUTCDATE())

1.8 TaskComments
Collaboration and updates on assigned action items.
- CommentId (INT, Primary Key, IDENTITY)
- TaskId (INT, Foreign Key -> Tasks.TaskId, NOT NULL)
- UserId (INT, Foreign Key -> Users.UserId, NOT NULL)
- CommentText (NVARCHAR(MAX), NOT NULL)
- CreatedAt (DATETIME2, NOT NULL, Default: GETUTCDATE())

1.9 Attachments
Metadata for MoM and Task file attachments.
- AttachmentId (INT, Primary Key, IDENTITY)
- EntityId (INT, NOT NULL) -- MoMId or TaskId
- EntityType (NVARCHAR(20), NOT NULL) -- MoM or Task
- FileName (NVARCHAR(255), NOT NULL)
- FilePath (NVARCHAR(MAX), NOT NULL)
- UploadedBy (INT, Foreign Key -> Users.UserId, NOT NULL)
- UploadedAt (DATETIME2, NOT NULL, Default: GETUTCDATE())

1.10 Notifications
Stores in-app notifications.
- NotificationId (INT, Primary Key, IDENTITY)
- UserId (INT, Foreign Key -> Users.UserId, NOT NULL)
- Title (NVARCHAR(150), NOT NULL)
- Message (NVARCHAR(MAX), NOT NULL)
- Type (NVARCHAR(50), NOT NULL)
- IsRead (BIT, NOT NULL, Default: 0)
- CreatedAt (DATETIME2, NOT NULL, Default: GETUTCDATE())

1.11 AuditLogs
Tracks activity for security and audit compliance.
- AuditLogId (INT, Primary Key, IDENTITY)
- UserId (INT, Foreign Key -> Users.UserId, NULLABLE)
- Action (NVARCHAR(100), NOT NULL)
- Details (NVARCHAR(MAX), NULLABLE)
- IpAddress (NVARCHAR(45), NULLABLE)
- Timestamp (DATETIME2, NOT NULL, Default: GETUTCDATE())

--------------------------------------------------------------------------------
2. INDEXES FOR PERFORMANCE OPTIMIZATION
--------------------------------------------------------------------------------
CREATE UNIQUE INDEX IX_Users_Email ON Users(Email);
CREATE INDEX IX_Meetings_Organizer_StartTime ON Meetings(OrganizerId, StartTime);
CREATE INDEX IX_Tasks_AssignedTo_Status ON Tasks(AssignedToUserId, Status);
CREATE INDEX IX_AuditLogs_UserId_Timestamp ON AuditLogs(UserId, Timestamp);
