# AURA — Database Design

**Version:** 1.0  
**Status:** Development Baseline  
**Database:** MySQL  
**Architecture:** Modular Monolith  
**Audience:** AI coding agents, developers, reviewers, and human supervisors

---

# 1. Purpose

This document defines the relational database design for AURA.

It is intended to be directly understandable by AI coding tools.

This document defines:

- Database responsibilities
- Tables
- Columns
- Data types
- Primary keys
- Foreign keys
- Relationships
- Ownership rules
- Indexing strategy
- Enum/status values
- Audit requirements
- Delete behavior
- Entity/DTO separation
- MySQL conventions
- AI implementation rules

The database is the **system of record for relational application data**.

The vector database is responsible for vector storage/search.

---

# 2. Source of Truth

Primary requirements come from:

```text
/docs/BRS.md
/docs/API.md
/docs/ARCHITECTURE.md
/docs/AI.md
```

The BRS defines MySQL as the relational database and specifies the initial relational entities including Users, Conversations, Messages, Tasks, Interactions, InteractionMessages, ActionItems, KnowledgeDocuments, KnowledgeChunks, and AgentMemories.

The selected AI stack also identifies MySQL as AURA's relational database and Qdrant as the vector database.

Where this document introduces implementation details that are not explicitly specified by the BRS, they are marked as:

```text
PROPOSED DECISION
```

These decisions may be changed before implementation if another project document requires it.

---

# 3. Database Responsibilities

MySQL stores:

```text
User information
User configuration
Conversations
Messages
Tasks
Interactions
Interaction messages
Action items
Knowledge document metadata
Knowledge chunk metadata
Agent memories
AI/tool audit information where required
```

MySQL does NOT store the primary vector representation for RAG.

Vector data belongs in:

```text
Qdrant
```

Conceptually:

```text
                 AURA Backend
                     |
          +----------+----------+
          |                     |
        MySQL                 Qdrant
          |                     |
   Relational data        Vector data
```

---

# 4. Database Design Principles

## 4.1 User Ownership

Every user-owned resource must be associated with a user.

Examples:

```text
Conversation -> User
Task         -> User
Interaction  -> User
Knowledge    -> User
Memory       -> User
```

The BRS explicitly requires user data isolation by identity.

---

# 5. User Identity Rule

The application obtains the current user from the authenticated identity.

The frontend must NOT control ownership.

Incorrect:

```text
POST /api/tasks

{
  "userId": "123",
  "title": "Review API"
}
```

Correct:

```text
POST /api/tasks

{
  "title": "Review API"
}
```

Backend:

```text
JWT
 ↓
Current User
 ↓
UserId
 ↓
Database operation
```

---

# 6. ID Strategy

## PROPOSED DECISION

Use UUIDs for application entity identifiers.

Example:

```text
550e8400-e29b-41d4-a716-446655440000
```

Reason:

- Suitable for distributed/application-generated identifiers
- Does not expose sequential record counts
- Works well with future integrations
- AI-generated references can treat IDs as opaque values

The frontend must never depend on the UUID format.

---

# 7. Timestamp Strategy

All persisted timestamps should use UTC.

Example:

```text
2026-09-10T08:30:00Z
```

Database/application convention:

```text
UTC
```

The frontend converts timestamps to the user's display timezone.

---

# 8. Table Overview

Initial relational model:

```text
Users
 |
 +-- Conversations
 |       |
 |       +-- Messages
 |
 +-- Tasks
 |
 +-- Interactions
 |       |
 |       +-- InteractionMessages
 |       |
 |       +-- ActionItems
 |
 +-- KnowledgeDocuments
 |       |
 |       +-- KnowledgeChunks
 |
 +-- AgentMemories
```

---

# 9. USERS

## Purpose

Stores the AURA user's identity/application profile and configuration.

Authentication is implemented using ASP.NET Core Identity + JWT.

The exact Identity implementation may introduce additional framework-managed tables.

This document focuses on the application-level user model.

---

## Table

```text
Users
```

### Columns

| Column | Type | Null | Key | Description |
|---|---|---:|---|---|
| Id | CHAR(36) | No | PK | User identifier |
| Name | VARCHAR(150) | No | | Display name |
| Email | VARCHAR(320) | No | UNIQUE | User email |
| Password/AuthReference | VARCHAR(...) | Depends | | Authentication reference |
| AgentName | VARCHAR(100) | Yes | | User's configured agent name |
| AvailabilityStatus | VARCHAR(30) | No | | Availability state |
| CreatedAt | DATETIME(6) | No | | Creation timestamp |
| UpdatedAt | DATETIME(6) | No | | Last update timestamp |

### Important

The exact password/authentication columns depend on the ASP.NET Core Identity implementation.

Do NOT create a custom password storage system if ASP.NET Core Identity is being used.

---

# 10. USER AVAILABILITY STATUS

Initial values:

```text
Available
Unavailable
RepresentativeMode
```

These values are derived from the representative-mode requirements in the BRS.

The application should use an enum rather than scattered string literals where practical.

---

# 11. CONVERSATIONS

## Purpose

Represents a conversation between the user and AURA.

The BRS requires conversation creation, text messaging, AI responses, and persisted conversation history.

---

## Table

```text
Conversations
```

### Columns

| Column | Type | Null | Key | Description |
|---|---|---:|---|---|
| Id | CHAR(36) | No | PK | Conversation identifier |
| UserId | CHAR(36) | No | FK | Owner |
| Title | VARCHAR(200) | Yes | | Conversation title |
| Status | VARCHAR(30) | No | | Conversation status |
| StartedAt | DATETIME(6) | No | | Start time |
| EndedAt | DATETIME(6) | Yes | | End time |
| CreatedAt | DATETIME(6) | No | | Creation time |
| UpdatedAt | DATETIME(6) | No | | Last modification |

### Relationship

```text
Users 1 ──────── * Conversations
```

---

# 12. CONVERSATION STATUS

Initial values:

```text
Active
Completed
Archived
```

## PROPOSED DECISION

The exact lifecycle may be simplified during implementation.

Do not create unnecessary states unless the application actually needs them.

---

# 13. CONVERSATION INDEXES

Required indexes:

```text
IX_Conversations_UserId
IX_Conversations_UserId_CreatedAt
```

Purpose:

```text
Retrieve user's conversations
Sort user's conversations by recent activity
```

---

# 14. MESSAGES

## Purpose

Stores messages belonging to conversations.

The BRS identifies message roles including User, Assistant, System, and Tool.

---

## Table

```text
Messages
```

### Columns

| Column | Type | Null | Key | Description |
|---|---|---:|---|---|
| Id | CHAR(36) | No | PK | Message identifier |
| ConversationId | CHAR(36) | No | FK | Parent conversation |
| Role | VARCHAR(30) | No | | Message role |
| Content | LONGTEXT | No | | Message content |
| MessageType | VARCHAR(30) | No | | Message type |
| CreatedAt | DATETIME(6) | No | | Creation timestamp |

---

# 15. MESSAGE ROLE

Initial values:

```text
User
Assistant
System
Tool
```

The backend controls these values.

The frontend must not be able to create arbitrary roles.

---

# 16. MESSAGE TYPE

Initial values from the BRS:

```text
Text
Voice
ToolResult
RAGResponse
```

The type may later be expanded if a real requirement exists.

---

# 17. MESSAGE RELATIONSHIP

```text
Conversation 1 ──────── * Messages
```

Foreign key:

```text
Messages.ConversationId
        ↓
Conversations.Id
```

---

# 18. MESSAGE INDEX

Required:

```text
IX_Messages_ConversationId_CreatedAt
```

Purpose:

```text
Load conversation history in chronological order.
```

---

# 19. TASKS

## Purpose

Stores user tasks.

Tasks may be created:

```text
Manually through UI
AI tool calling
n8n automation
```

The BRS requires create, retrieve, update, and complete task capabilities.

---

## Table

```text
Tasks
```

### Columns

| Column | Type | Null | Key | Description |
|---|---|---:|---|---|
| Id | CHAR(36) | No | PK | Task identifier |
| UserId | CHAR(36) | No | FK | Owner |
| Title | VARCHAR(250) | No | | Task title |
| Description | TEXT | Yes | | Task description |
| Priority | VARCHAR(30) | No | | Priority |
| DueDate | DATETIME(6) | Yes | | Due date |
| Status | VARCHAR(30) | No | | Task status |
| CreatedAt | DATETIME(6) | No | | Creation timestamp |
| UpdatedAt | DATETIME(6) | No | | Last update |

---

# 20. TASK PRIORITY

Initial values:

```text
Low
Medium
High
```

Future:

```text
Urgent
```

should only be added if required.

---

# 21. TASK STATUS

Initial values:

```text
Pending
Completed
```

Additional states may be introduced later if required.

---

# 22. TASK INDEXES

Required:

```text
IX_Tasks_UserId
IX_Tasks_UserId_Status
IX_Tasks_UserId_DueDate
```

These support:

```text
User task list
Pending task filtering
Tasks due today/tomorrow
```

---

# 23. INTERACTIONS

## Purpose

Stores interactions handled by AURA in Representative Mode.

The BRS requires interaction recording, transcript storage, summaries, action-item extraction, priority, and reporting.

---

## Table

```text
Interactions
```

### Columns

| Column | Type | Null | Key | Description |
|---|---|---:|---|---|
| Id | CHAR(36) | No | PK | Interaction identifier |
| UserId | CHAR(36) | No | FK | AURA owner |
| ParticipantName | VARCHAR(150) | No | | External participant |
| ParticipantContact | VARCHAR(250) | Yes | | Contact reference |
| Topic | VARCHAR(250) | Yes | | Interaction topic |
| Priority | VARCHAR(30) | Yes | | Interaction priority |
| Summary | TEXT | Yes | | AI-generated summary |
| StartedAt | DATETIME(6) | No | | Start time |
| EndedAt | DATETIME(6) | Yes | | End time |
| Status | VARCHAR(30) | No | | Interaction status |
| CreatedAt | DATETIME(6) | No | | Creation timestamp |
| UpdatedAt | DATETIME(6) | No | | Last update |

---

# 24. INTERACTION STATUS

Initial values:

```text
Active
Completed
Cancelled
```

---

# 25. INTERACTION PRIORITY

Initial values:

```text
Low
Medium
High
Urgent
```

The BRS requires interaction priority to be determined and displayed.

---

# 26. INTERACTION MESSAGES

## Purpose

Stores transcript messages for a representative interaction.

---

## Table

```text
InteractionMessages
```

### Columns

| Column | Type | Null | Key | Description |
|---|---|---:|---|---|
| Id | CHAR(36) | No | PK | Message identifier |
| InteractionId | CHAR(36) | No | FK | Parent interaction |
| Role | VARCHAR(30) | No | | Speaker role |
| Content | LONGTEXT | No | | Transcript content |
| CreatedAt | DATETIME(6) | No | | Timestamp |

---

# 27. INTERACTION MESSAGE ROLE

Initial values:

```text
Participant
Assistant
System
```

---

# 28. INTERACTION RELATIONSHIP

```text
Interaction 1 ──────── * InteractionMessages
```

Foreign key:

```text
InteractionMessages.InteractionId
        ↓
Interactions.Id
```

---

# 29. INTERACTION MESSAGE INDEX

Required:

```text
IX_InteractionMessages_InteractionId_CreatedAt
```

Purpose:

```text
Load transcript in chronological order.
```

---

# 30. ACTION ITEMS

## Purpose

Stores action items extracted from completed interactions.

The BRS requires action-item extraction and allows n8n to create tasks from action items.

---

## Table

```text
ActionItems
```

### Columns

| Column | Type | Null | Key | Description |
|---|---|---:|---|---|
| Id | CHAR(36) | No | PK | Action item identifier |
| InteractionId | CHAR(36) | No | FK | Source interaction |
| TaskId | CHAR(36) | Yes | FK | Created task, if applicable |
| Description | TEXT | No | | Action description |
| Priority | VARCHAR(30) | No | | Priority |
| Status | VARCHAR(30) | No | | Action item status |
| CreatedAt | DATETIME(6) | No | | Creation timestamp |
| UpdatedAt | DATETIME(6) | No | | Last update |

---

# 31. ACTION ITEM STATUS

Initial values:

```text
Pending
Completed
Dismissed
```

---

# 32. ACTION ITEM RELATIONSHIPS

```text
Interaction 1 ──────── * ActionItems

ActionItem * ───────── 0..1 Task
```

Meaning:

```text
An interaction can produce many action items.

An action item may optionally create/link to a task.
```

---

# 33. KNOWLEDGE DOCUMENTS

## Purpose

Stores metadata about documents uploaded by users.

The BRS requires users to upload supported documents, process them, generate embeddings, and make them searchable.

---

## Table

```text
KnowledgeDocuments
```

### Columns

| Column | Type | Null | Key | Description |
|---|---|---:|---|---|
| Id | CHAR(36) | No | PK | Document identifier |
| UserId | CHAR(36) | No | FK | Owner |
| FileName | VARCHAR(255) | No | | Original file name |
| FileType | VARCHAR(100) | No | | MIME/type |
| FilePath/StorageReference | VARCHAR(1000) | Yes | | Storage reference |
| Status | VARCHAR(30) | No | | Processing status |
| CreatedAt | DATETIME(6) | No | | Upload timestamp |
| ProcessedAt | DATETIME(6) | Yes | | Processing completion |

---

# 34. KNOWLEDGE DOCUMENT STATUS

Initial values:

```text
Uploading
Processing
Ready
Failed
```

These statuses are defined by the BRS.

---

# 35. DOCUMENT STORAGE RULE

MySQL should store metadata.

The actual file may be stored using:

```text
Local filesystem
OR
Object storage
```

The exact storage implementation is an infrastructure decision.

Therefore:

```text
FilePath/StorageReference
```

must be treated as an abstract storage reference.

Do not make application code dependent on a physical local filesystem path.

---

# 36. KNOWLEDGE CHUNKS

## Purpose

Stores extracted document chunks and metadata required to connect relational document information with Qdrant vectors.

---

## Table

```text
KnowledgeChunks
```

### Columns

| Column | Type | Null | Key | Description |
|---|---|---:|---|---|
| Id | CHAR(36) | No | PK | Chunk identifier |
| DocumentId | CHAR(36) | No | FK | Parent document |
| ChunkIndex | INT | No | | Chunk order |
| Content | LONGTEXT | No | | Extracted chunk |
| VectorReference | VARCHAR(255) | Yes | | Qdrant vector reference |
| MetadataJson | JSON | Yes | | Additional metadata |
| CreatedAt | DATETIME(6) | No | | Creation timestamp |

---

# 37. VECTOR STORAGE RULE

The vector itself should NOT be duplicated in MySQL unless there is a specific future requirement.

Architecture:

```text
KnowledgeChunks
      |
      | VectorReference
      v
Qdrant
      |
      v
Embedding/vector search
```

Qdrant stores the vector representation.

MySQL stores relational metadata.

---

# 38. KNOWLEDGE OWNERSHIP

Every knowledge document has:

```text
UserId
```

Retrieval must enforce:

```text
Authenticated User
       ↓
Allowed UserId
       ↓
Allowed DocumentIds
       ↓
Qdrant filter
```

The BRS explicitly requires RAG retrieval to enforce user/document ownership.

This is a mandatory security rule.

---

# 39. KNOWLEDGE CHUNK INDEXES

Required:

```text
IX_KnowledgeChunks_DocumentId
IX_KnowledgeChunks_DocumentId_ChunkIndex
```

---

# 40. AGENT MEMORIES

## Purpose

Stores non-sensitive user preferences/memory.

The BRS allows non-sensitive user preferences to be stored as memory and requires the user to be able to manage stored memory.

---

## Table

```text
AgentMemories
```

### Columns

| Column | Type | Null | Key | Description |
|---|---|---:|---|---|
| Id | CHAR(36) | No | PK | Memory identifier |
| UserId | CHAR(36) | No | FK | Owner |
| MemoryType | VARCHAR(50) | No | | Memory category |
| Content | TEXT | No | | Memory content |
| Importance | DECIMAL(4,3) | Yes | | Importance score |
| CreatedAt | DATETIME(6) | No | | Creation time |
| UpdatedAt | DATETIME(6) | No | | Last update |

---

# 41. MEMORY SAFETY

Only appropriate non-sensitive preferences should be stored.

Do NOT automatically store:

```text
Passwords
Authentication secrets
API keys
Payment information
Sensitive personal information
```

Memory storage must follow the project's AI safety rules.

---

# 42. MEMORY INDEX

Required:

```text
IX_AgentMemories_UserId
IX_AgentMemories_UserId_MemoryType
```

---

# 43. COMPLETE RELATIONSHIP MODEL

```text
                         Users
                           |
          +----------------+----------------+
          |                |                |
          v                v                v
   Conversations         Tasks       KnowledgeDocuments
          |                                |
          v                                v
       Messages                    KnowledgeChunks
                                           |
                                           |
                                        Qdrant
          
          |
          |
          +--------------------+
                               |
                               v
                         Interactions
                               |
                    +----------+----------+
                    |                     |
                    v                     v
          InteractionMessages       ActionItems
                                          |
                                          v
                                        Tasks

Users
  |
  v
AgentMemories
```

---

# 44. Foreign Key Summary

| Child Table | Column | Parent |
|---|---|---|
| Conversations | UserId | Users.Id |
| Messages | ConversationId | Conversations.Id |
| Tasks | UserId | Users.Id |
| Interactions | UserId | Users.Id |
| InteractionMessages | InteractionId | Interactions.Id |
| ActionItems | InteractionId | Interactions.Id |
| ActionItems | TaskId | Tasks.Id |
| KnowledgeDocuments | UserId | Users.Id |
| KnowledgeChunks | DocumentId | KnowledgeDocuments.Id |
| AgentMemories | UserId | Users.Id |

---

# 45. DELETE BEHAVIOR

## PROPOSED DECISION

Do not use unrestricted cascade deletion for all user data.

For example:

```text
Delete User
   X
   ↓
Automatically delete everything
```

should not be assumed.

Sensitive/destructive operations should be deliberate.

For individual resources:

```text
Delete Conversation
Delete Task
Delete Knowledge Document
```

the application should explicitly control dependent-data cleanup.

---

# 46. Knowledge Document Deletion

When a knowledge document is deleted:

```text
Verify ownership
      ↓
Remove/disable document
      ↓
Remove associated Qdrant vectors
      ↓
Remove relational chunks
      ↓
Return success
```

The database and vector store must not become permanently inconsistent.

If Qdrant deletion fails, the operation should be handled as a controlled failure and logged.

---

# 47. Conversation Deletion

Conversation deletion is not initially required by the BRS/API MVP.

Do not implement it merely because a DELETE endpoint is easy to add.

If introduced later, define:

```text
Transcript retention
Cascade behavior
AI audit retention
Vector references
```

before implementation.

---

# 48. Task Deletion

Tasks are explicitly deleteable through the API contract.

Deleting a task must NOT automatically delete the interaction/action item that referenced it.

Instead:

```text
ActionItem.TaskId
        ↓
NULL
```

if the task is removed.

This preserves the historical interaction record.

---

# 49. Interaction Retention

Completed interactions should remain available for reporting.

The BRS specifically requires users to review what happened while they were unavailable and retrieve interaction history.

Therefore, interaction records should not automatically disappear after n8n processing.

---

# 50. n8n Database Rule

n8n is an automation layer.

It should NOT become the source of truth for AURA business data.

Correct:

```text
AURA .NET
   ↓
MySQL
   ↓
n8n
```

n8n can read/process/create application data through approved APIs/services.

The BRS requires completed interactions to be processable by n8n and allows n8n to create tasks and notifications.

---

# 51. AI Tool Database Access

AI tools must NOT directly access MySQL.

Correct:

```text
LLM
 ↓
Tool Registry
 ↓
Application Service
 ↓
Repository
 ↓
MySQL
```

Incorrect:

```text
LLM
 ↓
SQL
 ↓
MySQL
```

This is a mandatory architecture/security rule.

---

# 52. Database Access Layer

.NET should use:

```text
Entity
 ↓
Repository/Data Access
 ↓
Application Service
```

Controllers should not contain SQL.

AI tools should not contain SQL.

Angular should never access MySQL.

---

# 53. Entity vs DTO

Database entities must not automatically become API responses.

Example:

```text
MySQL
 ↓
ConversationEntity
 ↓
Application
 ↓
ConversationResponseDto
 ↓
Angular
```

This protects the database schema from becoming the public API contract.

---

# 54. EF Core

## PROPOSED DECISION

Use Entity Framework Core for relational persistence unless a later project requirement establishes a reason not to.

Expected structure:

```text
AURA.Infrastructure
 |
 +-- Persistence
      |
      +-- AuraDbContext
      +-- Configurations
      +-- Repositories
      +-- Migrations
```

---

# 55. Database Migrations

Database schema changes should be version controlled.

Use EF Core migrations.

Conceptually:

```text
Entity change
      ↓
Migration
      ↓
Review
      ↓
Apply migration
      ↓
MySQL schema
```

AI coding tools must NOT manually modify production-like schema without updating migrations.

---

# 56. Naming Convention

Use:

```text
PascalCase
```

for C# entities/properties.

Use the project's selected MySQL naming convention consistently.

## PROPOSED DECISION

Prefer:

```text
Users
Conversations
Messages
Tasks
Interactions
InteractionMessages
ActionItems
KnowledgeDocuments
KnowledgeChunks
AgentMemories
```

The database table names should remain consistent with the API/domain terminology.

---

# 57. Nullability Rules

Nullable columns should represent genuine optional information.

Example:

```text
Conversation.EndedAt
```

can be NULL while a conversation is active.

Example:

```text
Task.DueDate
```

can be NULL because not every task needs a due date.

Do not make every column nullable simply to make insertion easier.

---

# 58. Required Application-Level Validation

Database constraints are not a replacement for API validation.

Example:

```text
API validation
+
Application business validation
+
Database constraints
```

All three may be required.

---

# 59. Example: Create Task

Request:

```json
{
  "title": "Review UCC validation",
  "description": "Review required validation rules.",
  "priority": "High",
  "dueDate": "2026-09-11T10:00:00Z"
}
```

Flow:

```text
Angular
 ↓
POST /api/tasks
 ↓
TaskController
 ↓
TaskApplicationService
 ↓
Validate request
 ↓
Get Current User
 ↓
Create Task Entity
 ↓
Repository
 ↓
MySQL
 ↓
TaskResponseDto
 ↓
Angular
```

---

# 60. Example: AI CreateTask

User:

```text
"Create a task to review UCC validation tomorrow."
```

Flow:

```text
Angular
 ↓
Conversation API
 ↓
Agent Orchestrator
 ↓
LLM
 ↓
CreateTask tool request
 ↓
Tool validation
 ↓
Task Application Service
 ↓
MySQL
 ↓
Tool result
 ↓
LLM
 ↓
Assistant response
 ↓
Messages
```

The BRS specifically describes this natural-language task flow.

---

# 61. Example: Knowledge Upload

```text
Angular
 ↓
POST /api/knowledge/documents
 ↓
.NET
 ↓
KnowledgeDocuments
 ↓
Document processing
 ↓
Extract text
 ↓
Chunk
 ↓
KnowledgeChunks
 ↓
BGE-M3 embedding
 ↓
Qdrant
 ↓
Document Status = Ready
```

The BRS defines the required extraction → chunking → embedding → vector storage flow.

---

# 62. Example: Interaction Completion

```text
Interaction
 ↓
InteractionMessages
 ↓
POST /api/interactions/{id}/complete
 ↓
Status = Completed
 ↓
Summary/action-item processing
 ↓
n8n
 ↓
Tasks/notifications
```

The interaction itself remains stored even if automation fails.

The BRS explicitly requires that n8n failure must not destroy the interaction record.

---

# 63. Auditability

The BRS requires AI actions to be auditable.

A dedicated audit table is therefore a likely future requirement.

## PROPOSED FUTURE TABLE

```text
AiActionLogs
```

Potential fields:

```text
Id
UserId
ConversationId
MessageId
ActionType
ToolName
InputSummary
ResultSummary
Success
CreatedAt
```

This table is intentionally NOT part of the initial schema until the audit implementation is designed.

Do not create it merely because it appears here.

---

# 64. Indexing Principle

Indexes should support actual queries.

Initial important access patterns:

```text
Get user's conversations
Get conversation messages
Get user's tasks
Get pending tasks
Get tasks by due date
Get user's interactions
Get interaction transcript
Get interaction action items
Get user's knowledge documents
Get document chunks
Get user's memories
```

Do not add indexes to every column.

---

# 65. Performance Principle

Initial database optimization should remain simple.

Do not introduce:

```text
Sharding
Read replicas
Partitioning
Database clusters
Complex caching
```

unless actual performance requirements justify them.

The BRS intentionally recommends a modular monolith and avoiding unnecessary infrastructure complexity.

---

# 66. Security Rules

AI coding agents implementing database functionality MUST follow:

1. Every user-owned entity must have ownership enforced.
2. Never trust `UserId` supplied by the frontend.
3. Always derive the current user from authentication.
4. Never allow cross-user conversation access.
5. Never allow cross-user task access.
6. Never allow cross-user interaction access.
7. Never allow cross-user knowledge retrieval.
8. Never allow cross-user memory access.
9. Validate foreign-key ownership relationships.
10. Never expose database credentials.
11. Never hardcode database passwords.
12. Never log database passwords.
13. Never log JWTs.
14. Never log API keys.

These rules support the BRS security requirements for identity isolation, authorized tools, RAG ownership, and protected APIs.

---

# 67. AI Coding Agent Implementation Checklist

Before modifying the database, the AI coding agent must:

```text
1. Read BRS.md.
2. Read ARCHITECTURE.md.
3. Read API.md.
4. Read DATABASE.md.
5. Identify the feature being implemented.
6. Identify affected entities.
7. Check existing relationships.
8. Check ownership requirements.
9. Check existing migrations.
10. Avoid unnecessary schema changes.
11. Update entity/configuration.
12. Create migration.
13. Update repository/application layer.
14. Update DTO/API if necessary.
15. Add validation.
16. Add tests.
17. Update documentation if the contract changed.
```

---

# 68. Schema Change Rule

If an AI coding agent wants to add a column:

```text
Do not simply add it.
```

It must first determine:

```text
Why is the column required?
Which requirement requires it?
Which API uses it?
Which feature uses it?
Is it already represented elsewhere?
Does it affect existing data?
Does it require a migration?
```

---

# 69. Example Schema Change

Suppose an AI agent wants:

```text
Tasks.CreatedByAi
```

It should determine whether the requirement actually needs an "AI-created indicator."

The BRS UI specification mentions an AI creation indicator for tasks.

Therefore this is a legitimate candidate for a future schema field.

However, the implementation must decide whether this should be:

```text
CreatedByAi BOOLEAN
```

or a more general:

```text
CreatedSource
```

That decision should be made before implementation rather than invented silently.

---

# 70. Database and Vector Database Boundary

```text
                    Knowledge
                       |
              +--------+--------+
              |                 |
             MySQL           Qdrant
              |                 |
        Metadata/data       Vectors/search
              |                 |
        DocumentId         VectorReference
        UserId             UserId
        FileName           DocumentId
        Status             ChunkIndex
        Chunk metadata     Embedding
```

Both systems must use compatible identifiers/metadata.

---

# 71. Database and SignalR Boundary

SignalR does not store the application state.

Example:

```text
Tool started
 ↓
SignalR event
```

This does NOT mean:

```text
SignalR = database
```

The authoritative tool execution result belongs to the backend/application and, where appropriate, persistent records.

SignalR is only the real-time communication channel.

---

# 72. Database and LLM Boundary

The LLM does not query MySQL directly.

Example:

```text
User:
"What tasks are due tomorrow?"

       ↓

LLM
       ↓
GetTasks tool
       ↓
Task Service
       ↓
MySQL
       ↓
Task result
       ↓
LLM
       ↓
Natural language answer
```

This keeps database authorization inside the application.

---

# 73. Initial Schema

The initial database contains:

```text
Users
Conversations
Messages
Tasks
Interactions
InteractionMessages
ActionItems
KnowledgeDocuments
KnowledgeChunks
AgentMemories
```

Potential future:

```text
AiActionLogs
Notifications
RefreshTokens
ExternalConnections
WorkflowExecutions
VoiceRecordings
```

Future tables must not be created until their corresponding feature is implemented.

---

# 74. MVP Database Scope

For the first runnable foundation, only create what is required for the current phase.

Phase 1:

```text
Users
```

and the minimum infrastructure required by authentication, if authentication is being implemented at that phase.

Phase 2:

```text
Conversations
Messages
```

Phase 3:

```text
Tasks
```

Phase 4:

```text
KnowledgeDocuments
KnowledgeChunks
```

Phase 5:

```text
Interactions
InteractionMessages
ActionItems
```

Phase 6:

```text
AgentMemories
```

This prevents premature database complexity.

---

# 75. Definition of Database Completion

A database feature is complete when:

```text
Requirement
    ↓
Entity
    ↓
Relationship
    ↓
Validation
    ↓
EF configuration
    ↓
Migration
    ↓
Database schema
    ↓
Repository/data access
    ↓
Application service
    ↓
API/AI integration
    ↓
Authorization
    ↓
Tests
    ↓
Documentation
```

A table existing in MySQL does NOT mean the feature is complete.

---

# 76. Final Database Architecture

```text
                         AURA
                           |
                     .NET 8 Backend
                           |
                  Application Services
                           |
             +-------------+-------------+
             |                           |
        Relational Data              Vector Data
             |                           |
             v                           v
          MySQL                        Qdrant
             |
    +--------+---------+
    |        |         |
    v        v         v
 Users   Conversations Tasks
            |
            v
         Messages

 Users
   |
   +---- Interactions
   |        |
   |        +---- InteractionMessages
   |        |
   |        +---- ActionItems
   |
   +---- KnowledgeDocuments
   |        |
   |        +---- KnowledgeChunks
   |
   +---- AgentMemories
```

---

# 77. Canonical Database Rule

> **MySQL is AURA's relational system of record. Qdrant is the vector-search system. Every user-owned resource must be isolated through authenticated user identity. AI tools never access MySQL directly. Database access occurs through the .NET application layer. API DTOs are separated from database entities. Schema changes require migrations and must be justified by an actual requirement.**

---

# 78. AI Coding Agent Final Instruction

When implementing database functionality:

```text
DO:
- Follow this schema.
- Preserve existing relationships.
- Enforce user ownership.
- Use migrations.
- Use typed entities.
- Use repositories/data-access abstractions.
- Keep business rules in the application layer.
- Test important operations.
- Update documentation when contracts change.

DO NOT:
- Invent unrelated tables.
- Add fields without a requirement.
- Allow frontend-controlled UserId.
- Let LLMs execute SQL.
- Let Angular access MySQL.
- Return EF entities directly from APIs.
- Store vectors in MySQL unnecessarily.
- Add Redis/Kafka/etc. for simple database operations.
- silently change existing relationships.
```

---

# 79. Status

```text
Database Architecture: Defined
Initial Entities: Defined
Relationships: Defined
Ownership Rules: Defined
Vector Boundary: Defined
Migration Strategy: Defined
Detailed SQL/EF Migration: Not yet generated
```

The actual EF Core migration should be generated **only when the corresponding implementation phase begins**.