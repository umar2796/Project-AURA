# AURA — API & Communication Contract

**Version:** 1.0  
**Status:** Development Baseline  
**Project:** AURA — AI Personal Representative  
**Audience:** AI coding agents, developers, reviewers, and human supervisors

---

# 1. Purpose

This document defines the communication contracts used by AURA.

It is intended to be consumed by both humans and AI coding tools.

This document defines:

- REST API endpoints
- HTTP methods
- Request JSON
- Response JSON
- Error responses
- Authentication requirements
- Validation rules
- Frontend ownership
- Angular service/component responsibilities
- Backend responsibilities
- Database interaction expectations
- AI/LLM interaction expectations
- RAG interaction expectations
- Tool-calling interaction expectations
- SignalR real-time events
- SignalR payloads
- Event naming conventions
- Request/response lifecycle
- Implementation constraints

The purpose is to ensure that different AI coding tools working on different parts of AURA produce compatible code.

---

# 2. Source of Truth

The following documents define AURA requirements:

```text
/docs/BRS.md
/docs/ARCHITECTURE.md
/docs/DATABASE.md
/docs/API.md
/docs/AI.md
```

Priority order:

```text
BRS
  ↓
Architecture
  ↓
Database
  ↓
API Contract
  ↓
AI Design
  ↓
Implementation
```

If an implementation conflicts with the BRS, the BRS takes priority.

If an implementation requires changing an API contract, the API document must be updated before changing the implementation.

AI coding tools must NOT silently invent new endpoints, request fields, response fields, database entities, or WebSocket events.

If something is not defined, the AI tool should identify the gap rather than inventing a major architectural decision.

---

# 3. Communication Architecture

AURA uses three primary communication mechanisms.

```text
                 ┌─────────────────────┐
                 │     Angular 17      │
                 │       Client       │
                 └──────────┬──────────┘
                            │
             ┌──────────────┴──────────────┐
             │                             │
          HTTP/REST                    SignalR
             │                             │
             ▼                             ▼
       ┌─────────────┐              ┌─────────────┐
       │  .NET API   │              │ SignalR Hub │
       └──────┬──────┘              └──────┬──────┘
              │                            │
              └────────────┬───────────────┘
                           ▼
                  ┌─────────────────┐
                  │ Application /   │
                  │ Agent Services  │
                  └───────┬─────────┘
                          │
        ┌─────────────────┼──────────────────┐
        ▼                 ▼                  ▼
      MySQL             AI Layer            n8n
                          │
                  ┌───────┼────────┐
                  ▼       ▼        ▼
                LLM      RAG     STT/TTS
```

---

# 4. Communication Responsibilities

## 4.1 HTTP / REST

HTTP is the default communication mechanism for:

- CRUD
- loading pages
- creating resources
- updating resources
- deleting resources
- uploading documents
- submitting messages
- retrieving history
- retrieving tasks
- retrieving interactions

HTTP should be used when the client does not require continuous real-time updates.

---

# 5. SignalR

ASP.NET Core SignalR is used for real-time application state and processing events.

SignalR is NOT the default replacement for REST.

Use:

```text
REST
→ Commands and queries

SignalR
→ Real-time events/status updates
```

Example:

```text
Angular
   |
   | POST message
   v
.NET API
   |
   | processing
   |
   +---- SignalR: agent.processing
   |
   +---- SignalR: rag.searching
   |
   +---- SignalR: tool.started
   |
   +---- SignalR: tool.completed
   |
   +---- SignalR: agent.responding
   |
   v
HTTP response
```

---

# 6. Base API URL

Development:

```text
/api
```

Example:

```text
GET /api/health
```

Production base URL must be configurable.

Angular must NOT hard-code production URLs inside components.

Use environment/configuration values.

---

# 7. Common HTTP Headers

Authenticated requests:

```http
Authorization: Bearer <JWT>
Content-Type: application/json
```

For file upload:

```http
Authorization: Bearer <JWT>
Content-Type: multipart/form-data
```

The Angular HTTP interceptor is responsible for attaching authentication information.

Components should not manually construct JWT headers.

---

# 8. Common Response Rules

Successful responses should return JSON.

JSON property naming convention:

```text
camelCase
```

Example:

```json
{
  "conversationId": "conv-123",
  "messageId": "msg-456",
  "createdAt": "2026-09-10T08:30:00Z"
}
```

Dates must use ISO-8601 format.

IDs should be treated as opaque identifiers by the frontend.

The frontend must not depend on database implementation details.

---

# 9. Common Error Contract

All API errors should follow a consistent structure.

Example:

```json
{
  "error": {
    "code": "CONVERSATION_NOT_FOUND",
    "message": "The requested conversation was not found.",
    "details": null,
    "traceId": "00-abcd1234"
  }
}
```

For validation:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "One or more validation errors occurred.",
    "details": {
      "title": [
        "Title is required."
      ]
    },
    "traceId": "00-abcd1234"
  }
}
```

The frontend should display user-friendly messages.

Internal exception details must not be exposed to users.

---

# 10. HTTP Status Code Guidelines

Use standard HTTP status codes.

| Status | Meaning |
|---|---|
| 200 | Successful request |
| 201 | Resource created |
| 204 | Successful request with no response body |
| 400 | Invalid request |
| 401 | Authentication required/invalid |
| 403 | User is authenticated but not authorized |
| 404 | Resource not found |
| 409 | Conflict |
| 422 | Validation/business-rule failure where appropriate |
| 500 | Unexpected server error |
| 503 | Dependency/service unavailable |

---

# 11. API Ownership

Each API must have a clear frontend feature owner.

Example:

```text
Conversation API
    ↓
ConversationService
    ↓
ChatComponent
```

The Angular component should NOT directly contain HTTP implementation.

Correct:

```text
ChatComponent
      ↓
ConversationService
      ↓
HttpClient
      ↓
.NET API
```

Incorrect:

```text
ChatComponent
      ↓
HttpClient
      ↓
.NET API
```

---

# 12. API MODULES

The initial API is divided into:

```text
Health
Conversations
Tasks
Knowledge
Agent
Voice
Interactions
Settings
Workflows
Authentication
```

Some endpoints may initially be implemented later according to the development phase.

---

# 13. HEALTH API

## 13.1 GET /api/health

### Purpose

Verify that the AURA backend is running.

### Authentication

Not required.

### Request

No body.

### Response

HTTP `200`

```json
{
  "status": "Healthy",
  "application": "AURA"
}
```

### Frontend

Used by:

```text
Health/Status component
Application startup verification
Development environment
```

### Angular service

```text
HealthService
```

### Backend

```text
HealthController
```

### Database

No database dependency.

### AI

No AI dependency.

### SignalR

No SignalR event.

---

# 14. CONVERSATION API

Conversations are the primary user-facing AI interaction.

---

## 14.1 POST /api/conversations

### Purpose

Create a new conversation.

### Authentication

Required.

### Request

```json
{
  "title": "UCC Validation Discussion"
}
```

### Fields

| Field | Type | Required |
|---|---|---|
| title | string | No |

If title is not supplied, the backend may assign a default title.

### Response

HTTP `201`

```json
{
  "conversationId": "conv-123",
  "title": "UCC Validation Discussion",
  "status": "Active",
  "startedAt": "2026-09-10T08:30:00Z"
}
```

### Frontend

```text
ConversationListComponent
ChatComponent
```

### Angular service

```text
ConversationService
```

### Database

Creates:

```text
Conversations
```

### AI

No LLM call required.

### SignalR

No event required.

---

# 15. GET /api/conversations

### Purpose

Retrieve conversations belonging to the authenticated user.

### Authentication

Required.

### Request

No body.

Optional query parameters may later be added for:

```text
page
pageSize
search
status
```

Do not implement pagination until it is required by the UI.

### Response

```json
{
  "items": [
    {
      "conversationId": "conv-123",
      "title": "UCC Validation Discussion",
      "status": "Active",
      "startedAt": "2026-09-10T08:30:00Z",
      "lastMessageAt": "2026-09-10T08:32:00Z"
    }
  ],
  "totalCount": 1
}
```

### Security

Only conversations belonging to the authenticated user may be returned.

The client must never provide a `userId` parameter to determine ownership.

The backend obtains the current user from the authenticated identity.

---

# 16. GET /api/conversations/{conversationId}

### Purpose

Retrieve one conversation and its messages.

### Authentication

Required.

### Response

```json
{
  "conversationId": "conv-123",
  "title": "UCC Validation Discussion",
  "status": "Active",
  "startedAt": "2026-09-10T08:30:00Z",
  "endedAt": null,
  "messages": [
    {
      "messageId": "msg-001",
      "role": "User",
      "content": "What validation is required?",
      "messageType": "Text",
      "createdAt": "2026-09-10T08:30:10Z"
    },
    {
      "messageId": "msg-002",
      "role": "Assistant",
      "content": "The required validation is...",
      "messageType": "Text",
      "createdAt": "2026-09-10T08:30:15Z"
    }
  ]
}
```

### Authorization

The requested conversation must belong to the authenticated user.

If not found or inaccessible:

```text
404
```

Do not expose whether another user's conversation exists.

---

# 17. POST /api/conversations/{conversationId}/messages

### Purpose

Send a user message to AURA.

This is one of the most important API contracts in the system.

### Authentication

Required.

### Request

```json
{
  "content": "Create a task to review UCC validation tomorrow.",
  "messageType": "Text"
}
```

### Fields

| Field | Type | Required |
|---|---|---|
| content | string | Yes |
| messageType | enum | Yes |

Initial message types:

```text
Text
Voice
```

### Processing

The backend must:

```text
Receive message
    ↓
Validate request
    ↓
Verify conversation ownership
    ↓
Persist user message
    ↓
Build conversation context
    ↓
Invoke Agent Orchestrator
    ↓
Agent decides:
    ├── Normal answer
    ├── RAG
    └── Tool call
    ↓
Persist assistant response
    ↓
Return response
```

### Response

```json
{
  "messageId": "msg-002",
  "conversationId": "conv-123",
  "role": "Assistant",
  "content": "Sure. I created a task to review UCC validation tomorrow.",
  "messageType": "Text",
  "createdAt": "2026-09-10T08:31:00Z"
}
```

### Important

The frontend must NOT directly call Ollama.

The frontend must NOT directly call Qdrant.

The frontend must NOT directly execute tools.

All AI orchestration happens inside .NET.

---

# 18. Message Processing SignalR Events

When processing a message, the backend may emit events.

Example sequence:

```text
message submitted
      ↓
agent.processing
      ↓
rag.searching       (only if RAG is required)
      ↓
tool.started        (only if tool is required)
      ↓
tool.completed      (only if tool is required)
      ↓
agent.responding
      ↓
conversation.completed
```

Not every request produces every event.

---

# 19. AGENT API

## 19.1 POST /api/agent/chat

### Purpose

Provide a direct AI-agent execution endpoint.

This endpoint is primarily a backend/application integration boundary.

The normal Angular chat UI should preferably use:

```text
POST /api/conversations/{conversationId}/messages
```

rather than directly depending on the agent endpoint.

### Request

```json
{
  "conversationId": "conv-123",
  "message": "What tasks are due tomorrow?"
}
```

### Response

```json
{
  "response": "You have 2 tasks due tomorrow.",
  "toolCalls": [
    {
      "toolName": "GetTasks",
      "success": true
    }
  ]
}
```

### Rule

The API controller must not contain agent reasoning logic.

Correct:

```text
Controller
   ↓
Application Service
   ↓
Agent Orchestrator
```

Incorrect:

```text
Controller
   ↓
Ollama API
```

---

# 20. TASK API

Tasks are resources that can also be manipulated through AI tool calling.

---

## 20.1 POST /api/tasks

### Request

```json
{
  "title": "Review UCC validation",
  "description": "Review required validation rules.",
  "priority": "High",
  "dueDate": "2026-09-11T10:00:00Z"
}
```

### Response

```json
{
  "taskId": "task-123",
  "title": "Review UCC validation",
  "description": "Review required validation rules.",
  "priority": "High",
  "dueDate": "2026-09-11T10:00:00Z",
  "status": "Pending",
  "createdAt": "2026-09-10T08:40:00Z"
}
```

### Database

```text
Tasks
```

### Frontend

```text
TaskListComponent
TaskFormComponent
TaskDetailsComponent
```

### Angular

```text
TaskService
```

---

# 21. GET /api/tasks

### Purpose

Retrieve tasks belonging to the current user.

### Response

```json
{
  "items": [
    {
      "taskId": "task-123",
      "title": "Review UCC validation",
      "priority": "High",
      "dueDate": "2026-09-11T10:00:00Z",
      "status": "Pending"
    }
  ],
  "totalCount": 1
}
```

---

# 22. GET /api/tasks/{taskId}

### Response

```json
{
  "taskId": "task-123",
  "title": "Review UCC validation",
  "description": "Review required validation rules.",
  "priority": "High",
  "dueDate": "2026-09-11T10:00:00Z",
  "status": "Pending",
  "createdAt": "2026-09-10T08:40:00Z",
  "updatedAt": "2026-09-10T08:40:00Z"
}
```

---

# 23. PUT /api/tasks/{taskId}

### Request

```json
{
  "title": "Review UCC validation",
  "description": "Review all UCC validation rules.",
  "priority": "High",
  "dueDate": "2026-09-12T10:00:00Z",
  "status": "Pending"
}
```

### Response

Return the updated task using the same task response structure.

---

# 24. DELETE /api/tasks/{taskId}

### Response

HTTP `204`

No response body.

### Rule

Deletion must respect authorization.

---

# 25. Task Tool Calling

The AI agent may use:

```text
CreateTask
GetTasks
UpdateTask
CompleteTask
```

The LLM must NEVER directly access MySQL.

Correct:

```text
LLM
 ↓
Tool Call
 ↓
Tool Registry
 ↓
Task Service
 ↓
Repository
 ↓
MySQL
```

---

# 26. KNOWLEDGE API

Knowledge APIs manage user-uploaded documents used by RAG.

---

# 27. POST /api/knowledge/documents

### Purpose

Upload a knowledge document.

### Authentication

Required.

### Request

Multipart form data:

```text
file = document
```

Supported formats should initially be limited to formats explicitly implemented by the ingestion service.

Initial target:

```text
PDF
TXT
MD
```

### Response

```json
{
  "documentId": "doc-123",
  "fileName": "UCC-BRS.pdf",
  "fileType": "application/pdf",
  "status": "Processing",
  "createdAt": "2026-09-10T09:00:00Z"
}
```

### Processing

```text
Upload
 ↓
Store document metadata
 ↓
Extract text
 ↓
Clean text
 ↓
Chunk
 ↓
Generate embeddings
 ↓
Store vectors in Qdrant
 ↓
Mark document Processed
```

Processing may be asynchronous.

---

# 28. GET /api/knowledge/documents

### Response

```json
{
  "items": [
    {
      "documentId": "doc-123",
      "fileName": "UCC-BRS.pdf",
      "fileType": "application/pdf",
      "status": "Processed",
      "createdAt": "2026-09-10T09:00:00Z",
      "processedAt": "2026-09-10T09:02:10Z"
    }
  ],
  "totalCount": 1
}
```

---

# 29. GET /api/knowledge/documents/{documentId}

### Response

```json
{
  "documentId": "doc-123",
  "fileName": "UCC-BRS.pdf",
  "fileType": "application/pdf",
  "status": "Processed",
  "createdAt": "2026-09-10T09:00:00Z",
  "processedAt": "2026-09-10T09:02:10Z"
}
```

---

# 30. DELETE /api/knowledge/documents/{documentId}

### Purpose

Delete a user's knowledge document and its associated vector data.

### Processing

```text
Verify ownership
 ↓
Delete/disable document
 ↓
Delete associated Qdrant vectors
 ↓
Return success
```

### Response

HTTP `204`

---

# 31. POST /api/knowledge/query

### Purpose

Perform a knowledge search.

This endpoint is primarily useful for explicit knowledge-search UI/testing.

Normal AI conversations should allow the Agent Orchestrator to invoke RAG automatically.

### Request

```json
{
  "query": "What validation is required for UCC?"
}
```

### Response

```json
{
  "query": "What validation is required for UCC?",
  "results": [
    {
      "documentId": "doc-123",
      "fileName": "UCC-BRS.pdf",
      "chunkIndex": 12,
      "content": "Relevant document content...",
      "score": 0.87
    }
  ]
}
```

### Security

Vector retrieval MUST filter by the authenticated user's allowed knowledge scope.

Minimum metadata:

```text
userId
documentId
fileName
chunkIndex
```

A user must never retrieve another user's vectors.

---

# 32. RAG SignalR Event

When RAG is used during an agent request:

```json
{
  "event": "rag.searching",
  "conversationId": "conv-123",
  "messageId": "msg-456"
}
```

Do not send private retrieved document content through the event unless explicitly required by the UI.

---

# 33. VOICE API

Voice processing is introduced after the text agent is stable.

---

# 34. POST /api/voice/transcribe

### Purpose

Convert uploaded audio into text.

### Request

Multipart form data:

```text
audio = audio file/blob
```

### Response

```json
{
  "text": "Create a task to review UCC validation tomorrow.",
  "language": "en",
  "durationMs": 4200
}
```

### Technology

```text
faster-whisper
```

The Angular application must not directly depend on the STT implementation.

---

# 35. POST /api/voice/synthesize

### Purpose

Convert text into speech.

### Request

```json
{
  "text": "Sure, I have created the task.",
  "voice": "default"
}
```

### Response

The exact transport format may be binary audio or an audio resource depending on the implementation.

Initial contract may return:

```json
{
  "audioId": "audio-123",
  "contentType": "audio/wav"
}
```

The implementation must document the actual audio delivery mechanism before frontend integration.

### Technology

```text
Piper
```

---

# 36. Voice Conversation Flow

Initial MVP voice flow:

```text
Angular microphone
       ↓
Audio capture
       ↓
POST /api/voice/transcribe
       ↓
Text
       ↓
POST /api/conversations/{id}/messages
       ↓
Agent
       ↓
Text response
       ↓
POST /api/voice/synthesize
       ↓
Audio
       ↓
Angular playback
```

Do NOT implement full-duplex streaming until the basic pipeline works.

---

# 37. Voice SignalR Events

The UI may receive:

### voice.listening

```json
{
  "event": "voice.listening",
  "conversationId": "conv-123"
}
```

### voice.transcribing

```json
{
  "event": "voice.transcribing",
  "conversationId": "conv-123"
}
```

### voice.speaking

```json
{
  "event": "voice.speaking",
  "conversationId": "conv-123",
  "messageId": "msg-456"
}
```

---

# 38. INTERACTION API

Interactions represent external-person conversations with AURA in Representative Mode.

---

# 39. POST /api/interactions

### Purpose

Create a representative-mode interaction.

### Request

```json
{
  "participantName": "Ahmed",
  "participantContact": "optional-contact",
  "topic": null
}
```

### Response

```json
{
  "interactionId": "interaction-123",
  "participantName": "Ahmed",
  "status": "Active",
  "startedAt": "2026-09-10T10:00:00Z"
}
```

---

# 40. GET /api/interactions

### Purpose

Retrieve interactions for the authenticated user.

### Response

```json
{
  "items": [
    {
      "interactionId": "interaction-123",
      "participantName": "Ahmed",
      "topic": "Production API issue",
      "priority": "High",
      "status": "Completed",
      "startedAt": "2026-09-10T10:00:00Z",
      "endedAt": "2026-09-10T10:04:32Z"
    }
  ],
  "totalCount": 1
}
```

---

# 41. GET /api/interactions/{interactionId}

### Response

```json
{
  "interactionId": "interaction-123",
  "participantName": "Ahmed",
  "participantContact": null,
  "topic": "Production API issue",
  "priority": "High",
  "status": "Completed",
  "startedAt": "2026-09-10T10:00:00Z",
  "endedAt": "2026-09-10T10:04:32Z",
  "summary": "Participant reported HTTP 500 errors from the production API.",
  "messages": [
    {
      "messageId": "imsg-001",
      "role": "Participant",
      "content": "The production API is returning 500 errors.",
      "createdAt": "2026-09-10T10:01:00Z"
    },
    {
      "messageId": "imsg-002",
      "role": "Assistant",
      "content": "When did the issue start?",
      "createdAt": "2026-09-10T10:01:05Z"
    }
  ],
  "actionItems": [
    {
      "actionItemId": "action-001",
      "description": "Investigate production API errors.",
      "priority": "High",
      "status": "Pending"
    }
  ]
}
```

---

# 42. POST /api/interactions/{interactionId}/complete

### Purpose

Complete an interaction.

### Request

```json
{}
```

### Processing

```text
Complete interaction
      ↓
Persist final transcript
      ↓
Trigger interaction-completed processing
      ↓
n8n webhook
      ↓
Summary/action-item processing
```

### Response

```json
{
  "interactionId": "interaction-123",
  "status": "Completed",
  "endedAt": "2026-09-10T10:04:32Z"
}
```

---

# 43. WORKFLOW API

## POST /api/workflows/interaction-completed

### Purpose

Trigger n8n processing for a completed interaction.

### Internal/System Endpoint

This endpoint must NOT be publicly callable without appropriate authentication or secret validation.

### Request

```json
{
  "interactionId": "interaction-123",
  "eventType": "interaction.completed"
}
```

### Expected n8n processing

```text
Receive interaction ID
       ↓
Get interaction
       ↓
Generate summary
       ↓
Extract action items
       ↓
Create tasks where appropriate
       ↓
Notify user
```

The interaction must remain stored even if n8n is unavailable.

Automation failure must not destroy the source interaction.

---

# 44. SETTINGS API

## GET /api/settings

### Purpose

Retrieve current AURA configuration for the authenticated user.

### Response

```json
{
  "agentName": "AURA",
  "availabilityStatus": "Available",
  "representativeModeEnabled": false,
  "defaultVoice": "default"
}
```

---

# 45. PUT /api/settings

### Request

```json
{
  "agentName": "AURA",
  "availabilityStatus": "Available",
  "representativeModeEnabled": false,
  "defaultVoice": "default"
}
```

### Rule

Settings must belong to the authenticated user.

---

# 46. AUTHENTICATION

Authentication is implemented using:

```text
ASP.NET Core Identity
+
JWT
```

Protected APIs require:

```http
Authorization: Bearer <JWT>
```

The exact authentication endpoints should be defined when the authentication phase begins.

Initial public endpoint:

```text
GET /api/health
```

All user data APIs should eventually require authentication.

---

# 47. USER OWNERSHIP RULE

This is a critical security rule.

The frontend must NOT decide ownership.

Incorrect:

```json
{
  "userId": "123",
  "conversationId": "456"
}
```

Correct:

```json
{
  "conversationId": "456"
}
```

The backend determines:

```text
Current authenticated user
        ↓
UserId
        ↓
Resource ownership
```

Every query for user-owned resources must apply ownership/authorization.

---

# 48. SIGNALR CONTRACT

## 48.1 Purpose

SignalR provides real-time state updates to the Angular UI.

Initial event list:

```text
conversation.started
voice.listening
voice.transcribing
agent.processing
rag.searching
tool.started
tool.completed
agent.responding
voice.speaking
conversation.completed
error
```

These event names are part of the AURA communication contract.

---

# 49. Standard SignalR Event Structure

Events should use a consistent envelope.

Example:

```json
{
  "event": "agent.processing",
  "conversationId": "conv-123",
  "messageId": "msg-456",
  "timestamp": "2026-09-10T10:00:00Z",
  "data": {}
}
```

Fields:

| Field | Required |
|---|---|
| event | Yes |
| conversationId | Usually |
| messageId | When related to message |
| timestamp | Yes |
| data | Yes |

The exact event-specific fields belong inside `data` where practical.

---

# 50. conversation.started

```json
{
  "event": "conversation.started",
  "conversationId": "conv-123",
  "timestamp": "2026-09-10T10:00:00Z",
  "data": {
    "status": "Active"
  }
}
```

---

# 51. agent.processing

```json
{
  "event": "agent.processing",
  "conversationId": "conv-123",
  "messageId": "msg-456",
  "timestamp": "2026-09-10T10:00:01Z",
  "data": {
    "status": "Processing"
  }
}
```

---

# 52. rag.searching

```json
{
  "event": "rag.searching",
  "conversationId": "conv-123",
  "messageId": "msg-456",
  "timestamp": "2026-09-10T10:00:02Z",
  "data": {
    "status": "Searching knowledge"
  }
}
```

Do not send private document content through this event.

---

# 53. tool.started

```json
{
  "event": "tool.started",
  "conversationId": "conv-123",
  "messageId": "msg-456",
  "timestamp": "2026-09-10T10:00:03Z",
  "data": {
    "toolName": "CreateTask"
  }
}
```

---

# 54. tool.completed

```json
{
  "event": "tool.completed",
  "conversationId": "conv-123",
  "messageId": "msg-456",
  "timestamp": "2026-09-10T10:00:04Z",
  "data": {
    "toolName": "CreateTask",
    "success": true
  }
}
```

Do not expose sensitive internal tool execution details.

---

# 55. agent.responding

```json
{
  "event": "agent.responding",
  "conversationId": "conv-123",
  "messageId": "msg-456",
  "timestamp": "2026-09-10T10:00:05Z",
  "data": {
    "status": "Generating response"
  }
}
```

---

# 56. conversation.completed

```json
{
  "event": "conversation.completed",
  "conversationId": "conv-123",
  "messageId": "msg-456",
  "timestamp": "2026-09-10T10:00:06Z",
  "data": {
    "status": "Completed"
  }
}
```

---

# 57. error

```json
{
  "event": "error",
  "conversationId": "conv-123",
  "messageId": "msg-456",
  "timestamp": "2026-09-10T10:00:07Z",
  "data": {
    "code": "AI_SERVICE_UNAVAILABLE",
    "message": "AURA is temporarily unable to process the request."
  }
}
```

Do not expose stack traces or internal exception information.

---

# 58. Angular SignalR Responsibility

Angular should have a dedicated service:

```text
SignalRService
```

Example conceptual flow:

```text
SignalRService
      ↓
RxJS Observable
      ↓
Feature component/service
      ↓
UI state
```

Components should not contain low-level SignalR connection management.

---

# 59. UI STATE MODEL

The frontend may represent agent processing states as:

```text
IDLE
LISTENING
TRANSCRIBING
THINKING
RAG_SEARCHING
TOOL_EXECUTING
RESPONDING
SPEAKING
ERROR
```

Example mapping:

```text
voice.listening
      ↓
LISTENING

voice.transcribing
      ↓
TRANSCRIBING

agent.processing
      ↓
THINKING

rag.searching
      ↓
RAG_SEARCHING

tool.started
      ↓
TOOL_EXECUTING

agent.responding
      ↓
RESPONDING

voice.speaking
      ↓
SPEAKING

error
      ↓
ERROR
```

The frontend should not expose unnecessary internal implementation states.

---

# 60. END-TO-END TEXT CHAT FLOW

Example user message:

```text
"Create a task to review UCC validation tomorrow."
```

Flow:

```text
Angular ChatComponent
        |
        v
ConversationService
        |
        | POST
        v
.NET ConversationController
        |
        v
Conversation Application Service
        |
        v
Persist User Message
        |
        v
Agent Orchestrator
        |
        +---- SignalR agent.processing
        |
        v
LLM
        |
        v
CreateTask tool call
        |
        +---- SignalR tool.started
        |
        v
Tool Registry
        |
        v
Task Service
        |
        v
MySQL
        |
        +---- SignalR tool.completed
        |
        v
LLM
        |
        v
Final response
        |
        +---- SignalR agent.responding
        |
        v
Persist Assistant Message
        |
        +---- SignalR conversation.completed
        |
        v
HTTP Response
        |
        v
Angular ChatComponent
```

---

# 61. END-TO-END RAG FLOW

Example:

```text
"What does my uploaded UCC document say about validation?"
```

Flow:

```text
Angular
   |
   v
Conversation API
   |
   v
Agent Orchestrator
   |
   v
LLM determines knowledge is required
   |
   +---- SignalR rag.searching
   |
   v
Embedding Service
   |
   v
BGE-M3
   |
   v
Qdrant
   |
   +---- UserId filter
   |
   v
Top-K chunks
   |
   v
Agent
   |
   v
LLM
   |
   v
Grounded response
   |
   v
Angular
```

The LLM must receive only authorized retrieved context.

---

# 62. END-TO-END VOICE FLOW

```text
User
 ↓
Angular microphone
 ↓
Audio
 ↓
POST /api/voice/transcribe
 ↓
faster-whisper
 ↓
Text
 ↓
POST /api/conversations/{id}/messages
 ↓
Agent
 ↓
Text response
 ↓
POST /api/voice/synthesize
 ↓
Piper
 ↓
Audio
 ↓
Angular
 ↓
Speaker
```

SignalR provides processing state updates.

---

# 63. END-TO-END REPRESENTATIVE FLOW

```text
External Participant
        ↓
Representative UI
        ↓
Interaction API
        ↓
Interaction created
        ↓
Conversation with AURA
        ↓
STT / Agent / RAG / Tools
        ↓
Interaction transcript
        ↓
Interaction completed
        ↓
n8n webhook
        ↓
Summary
        ↓
Action items
        ↓
Tasks / Notification
        ↓
User reviews interaction
```

---

# 64. AI PROVIDER BOUNDARY

The API layer must not depend directly on Ollama-specific request structures.

Use application interfaces.

```text
IAiChatService
IAiEmbeddingService
ISttService
ITtsService
IVectorStore
IN8nService
```

Architecture:

```text
Application
    |
    +-- IAiChatService
    |
    +-- IAiEmbeddingService
    |
    +-- ISttService
    |
    +-- ITtsService
    |
    +-- IVectorStore
    |
    +-- IN8nService
             |
             v
       Infrastructure
```

Current implementations:

```text
IAiChatService
      ↓
Ollama / Qwen3 8B

IAiEmbeddingService
      ↓
BGE-M3

ISttService
      ↓
faster-whisper

ITtsService
      ↓
Piper

IVectorStore
      ↓
Qdrant

IN8nService
      ↓
self-hosted n8n
```

The current technology stack is defined in the AURA AI Stack document.

---

# 65. LLM TOOL CALLING CONTRACT

The LLM may request a tool.

Example conceptual structure:

```json
{
  "toolName": "CreateTask",
  "arguments": {
    "title": "Review UCC validation",
    "description": "Review required validation rules.",
    "priority": "High",
    "dueDate": "2026-09-11T10:00:00Z"
  }
}
```

The backend must:

```text
Receive tool request
      ↓
Validate tool name
      ↓
Validate arguments
      ↓
Check current user
      ↓
Check authorization
      ↓
Check confirmation requirement
      ↓
Execute application service
      ↓
Return structured result
```

The LLM does not get direct database access.

---

# 66. TOOL RESULT CONTRACT

Example:

```json
{
  "success": true,
  "toolName": "CreateTask",
  "result": {
    "taskId": "task-123",
    "status": "Created"
  },
  "error": null
}
```

Failure:

```json
{
  "success": false,
  "toolName": "CreateTask",
  "result": null,
  "error": {
    "code": "TASK_CREATION_FAILED",
    "message": "The task could not be created."
  }
}
```

The LLM should receive structured tool results and generate the final user-facing response.

---

# 67. CONFIRMATION RULE

Potentially risky actions may require confirmation.

Example:

```text
User:
"Delete all my tasks."

Agent:
"This will permanently delete your tasks. Do you want me to continue?"
```

The tool must not execute until confirmation is satisfied.

The exact confirmation mechanism will be implemented during the security/polish phase.

---

# 68. API ↔ DATABASE RULE

The API must not expose database entities directly.

Use DTOs.

Correct:

```text
Database Entity
      ↓
Application
      ↓
DTO
      ↓
API Response
```

Do not return EF Core entities directly from controllers.

---

# 69. API ↔ FRONTEND RULE

Angular interfaces should represent API contracts.

Example:

```text
ConversationResponse
MessageResponse
TaskResponse
InteractionResponse
KnowledgeDocumentResponse
ApiError
SignalREvent
```

The frontend should not recreate API response structures independently.

The API document is the source of truth.

---

# 70. Naming Conventions

REST:

```text
/api/conversations
/api/tasks
/api/knowledge/documents
/api/interactions
```

Use plural nouns for resources.

Avoid:

```text
/api/getConversations
/api/createTask
/api/deleteTask
```

HTTP method expresses the operation.

---

# 71. Versioning

Initial API:

```text
/api/...
```

Do not introduce:

```text
/api/v1/
```

unless versioning becomes necessary.

If API versioning is later introduced, all affected contracts must be updated.

---

# 72. Pagination

Pagination is not mandatory for the first implementation.

When required, use a consistent structure:

```json
{
  "items": [],
  "totalCount": 0,
  "page": 1,
  "pageSize": 20
}
```

Do not introduce different pagination formats for different modules.

---

# 73. API Implementation Rules for AI Coding Agents

An AI coding tool implementing an API must follow these rules:

1. Read `BRS.md`.
2. Read `ARCHITECTURE.md`.
3. Read `DATABASE.md`.
4. Read this `API.md`.
5. Read `AI.md` when implementing AI functionality.
6. Do not invent API contracts without documenting them.
7. Do not expose database entities directly.
8. Use DTOs.
9. Validate input.
10. Validate resource ownership.
11. Use dependency injection.
12. Keep controllers thin.
13. Keep business logic outside controllers.
14. Do not call external AI providers directly from controllers.
15. Do not allow LLM direct database access.
16. Use interfaces for external AI/infrastructure providers.
17. Follow the existing error contract.
18. Follow the existing naming conventions.
19. Add tests for important business logic.
20. Update this document when the contract intentionally changes.

---

# 74. Frontend Implementation Rules for AI Coding Agents

An Angular coding tool must:

1. Read the API contract before implementing API integration.
2. Use Angular services for HTTP communication.
3. Use typed request/response interfaces.
4. Use an HTTP interceptor for authentication.
5. Use the SignalR service for real-time events.
6. Keep API URLs in configuration/environment.
7. Do not call Ollama directly.
8. Do not call Qdrant directly.
9. Do not call n8n directly unless an explicitly defined frontend requirement exists.
10. Do not place business logic inside presentation components unnecessarily.
11. Handle loading states.
12. Handle API errors.
13. Handle SignalR connection errors.
14. Do not invent response fields.
15. Keep UI state separate from backend DTOs when appropriate.

---

# 75. Backend Implementation Rules for AI Coding Agents

A .NET coding tool must:

```text
Controller
   ↓
Application Service / Use Case
   ↓
Domain / Business Logic
   ↓
Infrastructure
   ↓
External System / Database
```

Do not implement:

```text
Controller
   ↓
Repository
```

for complex business operations.

Controllers should primarily:

```text
Receive request
Validate basic HTTP concerns
Call application service
Return response
```

---

# 76. API Change Procedure

If an AI tool discovers that an existing API contract is insufficient:

### Step 1

Identify the limitation.

### Step 2

Check whether the BRS already defines the requirement.

### Step 3

Check Architecture and Database documents.

### Step 4

Propose the API contract change.

### Step 5

Update `API.md`.

### Step 6

Update affected Angular/.NET contracts.

### Step 7

Implement.

Do not silently change the contract.

---

# 77. Current MVP API Priority

The implementation order is:

## Phase 1

```text
GET /api/health
```

## Phase 2

```text
POST /api/conversations
GET /api/conversations
GET /api/conversations/{id}
POST /api/conversations/{id}/messages
```

## Phase 3

```text
POST /api/tasks
GET /api/tasks
GET /api/tasks/{id}
PUT /api/tasks/{id}
DELETE /api/tasks/{id}
```

## Phase 4

```text
Knowledge APIs
RAG
```

## Phase 5

```text
Voice APIs
SignalR
```

## Phase 6

```text
Interaction APIs
Representative Mode
```

## Phase 7

```text
n8n workflow integration
```

## Phase 8

```text
Authentication
Authorization
Error handling
Observability
Polish
```

This follows the incremental development principle in the BRS: foundation first, then text AI, agent, RAG, voice, representative mode, automation, and polish.

---

# 78. API Contract Status

| Module | Contract Status | Implementation |
|---|---|---|
| Health | Defined | Phase 1 |
| Conversations | Defined | Phase 2 |
| Tasks | Defined | Phase 3 |
| Knowledge | Defined | Phase 4 |
| Agent | Defined | Phase 2/3 |
| Voice | Initial contract | Phase 5 |
| SignalR | Initial contract | Phase 5 |
| Interactions | Defined | Phase 6 |
| Workflows | Initial contract | Phase 7 |
| Settings | Defined | Phase 8 |
| Authentication | Initial direction | Phase 8 |

---

# 79. Important Contract Limitations

The following are intentionally NOT fully specified yet:

- Streaming LLM responses
- Streaming STT
- Streaming TTS
- Full-duplex voice
- Real phone communication
- WhatsApp
- Email
- Calendar
- External SaaS integrations
- Multi-user SaaS APIs
- Advanced analytics
- Complex notification providers

These are future extensions and must not block the MVP.

---

# 80. Final API Architecture

The final conceptual communication architecture is:

```text
                         AURA
                          |
              ┌───────────┴───────────┐
              |                       |
           Angular                  External
              |                    Participant
              |                       |
       ┌──────┴──────┐                |
       |             |                |
      HTTP        SignalR             |
       |             |                |
       └──────┬──────┘                |
              ▼                       |
        .NET 8 Backend <──────────────┘
              |
       Application Layer
              |
       ┌──────┼────────┬───────────┐
       ▼      ▼        ▼           ▼
     MySQL   Agent    RAG       Workflow
              |        |           |
              ▼        ▼           ▼
            Ollama   Qdrant       n8n
              |
        ┌─────┼─────┐
        ▼     ▼     ▼
       LLM   STT   TTS
```

---

# 81. Canonical Rule

When implementing AURA, remember:

> **REST is for commands and queries. SignalR is for real-time events. .NET owns business logic and orchestration. MySQL owns relational application data. Qdrant owns vector search. The AI layer is accessed through provider abstractions. The LLM never directly accesses application databases or tools. Angular communicates with AURA through defined contracts rather than directly communicating with infrastructure services.**

---

# 82. Definition of API Completion

An API feature is considered complete only when:

```text
Requirement
    ↓
API Contract
    ↓
DTO
    ↓
Validation
    ↓
Controller
    ↓
Application Service
    ↓
Domain/Business Logic
    ↓
Infrastructure
    ↓
Database/External Service
    ↓
Error Handling
    ↓
Logging where appropriate
    ↓
Tests
    ↓
Angular integration where applicable
    ↓
SignalR integration where applicable
    ↓
Documentation updated
```

Compilation alone does NOT mean an API feature is complete.

---

# 83. AI Coding Agent Instruction

Before modifying AURA code, an AI coding agent should answer internally:

```text
1. Which BRS requirement am I implementing?
2. Which API contract applies?
3. Which Angular component/service owns the client side?
4. Which .NET layer owns the business logic?
5. Which database entities are involved?
6. Does AI/RAG/tool calling participate?
7. Does SignalR need to emit an event?
8. What authentication/authorization applies?
9. What happens on failure?
10. Does the API contract need updating?
```

If any answer is unclear, the agent should inspect the project documentation before implementing.

---

# 84. Final Contract Principle

AURA is being developed through multiple AI coding tools under human supervision.

Therefore:

```text
Documentation
      ↓
Shared Contract
      ↓
AI Coding Tool
      ↓
Implementation
      ↓
Human Review
```

The documentation is not optional supporting material.

**The documentation acts as the shared engineering context between AI coding agents.**

Any AI tool working on AURA must treat these documents as project-level instructions and must preserve compatibility with existing contracts.