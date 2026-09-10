# AURA — System Architecture

**Version:** 1.0  
**Status:** Development Baseline  
**Project:** AURA — AI Personal Representative  
**Architecture Style:** Modular Monolith  
**Audience:** AI coding agents, developers, reviewers, and human supervisors

---

# 1. Purpose

This document defines the technical architecture of AURA.

It is written so that multiple AI coding tools can independently implement different parts of AURA while maintaining a consistent architecture.

For example:

```text
Claude
  ↓
Angular

Another AI tool
  ↓
n8n / workflow

AI coding assistant
  ↓
.NET backend

Another AI tool
  ↓
AI/RAG components
```

All implementation agents must use this document together with:

```text
/docs/BRS.md
/docs/API.md
/docs/DATABASE.md
/docs/AI.md
```

as the shared engineering context.

---

# 2. Architectural Goals

AURA architecture must provide:

1. Clear separation of concerns.
2. Easy replacement of AI providers.
3. Secure user-data isolation.
4. Testable backend logic.
5. Clear frontend/backend contracts.
6. Real-time communication support.
7. RAG integration.
8. Tool/function calling.
9. Voice integration.
10. n8n automation.
11. Future extensibility without unnecessary complexity.
12. Simple local development.
13. Strong portfolio/interview value.

The architecture should demonstrate enterprise engineering practices without introducing infrastructure merely for appearance.

---

# 3. Architectural Style

AURA uses a:

> **Modular Monolith with Clean Architecture-style separation.**

This means AURA is deployed initially as one main backend application, but its internal responsibilities are clearly separated.

Conceptually:

```text
AURA
 |
 +-- Angular Client
 |
 +-- .NET Backend
       |
       +-- API
       +-- Application
       +-- Domain
       +-- Infrastructure
       |
       +-- AI
       +-- RAG
       +-- Voice
       +-- Tools
       +-- Automation
       |
       +-- MySQL
       +-- Qdrant
       +-- n8n
```

The BRS explicitly recommends a modular monolith and warns against prematurely introducing Kubernetes, microservices, Kafka, complex event buses, excessive agents, or unnecessary infrastructure.

---

# 4. High-Level System Architecture

```text
                              ┌───────────────────┐
                              │    AURA User      │
                              └─────────┬─────────┘
                                        │
                                  Browser / UI
                                        │
                              ┌─────────▼─────────┐
                              │   Angular 17      │
                              │     Client        │
                              └──────┬─────┬──────┘
                                     │     │
                                  HTTP   SignalR
                                     │     │
                              ┌──────▼─────▼──────┐
                              │      .NET 8       │
                              │      Backend      │
                              └─────────┬──────────┘
                                        │
                         ┌──────────────┼──────────────┐
                         │              │              │
                         ▼              ▼              ▼
                    Application      Agent          APIs
                         │              │
                         │              ├── LLM
                         │              ├── RAG
                         │              └── Tools
                         │
              ┌──────────┼───────────┐
              │          │           │
              ▼          ▼           ▼
            MySQL     Qdrant        n8n
                                     
                         AI Infrastructure
                              │
                   ┌──────────┼──────────┐
                   ▼          ▼          ▼
                 Ollama    STT/TTS     Embeddings
                 Qwen3      Whisper      BGE-M3
                             Piper
```

---

# 5. Major Components

| Component | Responsibility |
|---|---|
| Angular 17 | User interface |
| .NET 8 API | Application entry point |
| Application Layer | Business use cases |
| Domain Layer | Core business rules |
| Infrastructure Layer | External systems/persistence |
| Agent Orchestrator | AI reasoning workflow |
| Tool Registry | AI tool execution |
| RAG Service | Knowledge retrieval |
| Voice Service | STT/TTS integration |
| SignalR Hub | Real-time events |
| MySQL | Relational system of record |
| Qdrant | Vector storage/search |
| Ollama | Local LLM runtime |
| faster-whisper | Local STT |
| Piper | Local TTS |
| BGE-M3 | Embeddings |
| n8n | Workflow automation |

---

# 6. Frontend Architecture

AURA uses Angular 17.

The BRS recommends:

- Feature-based architecture
- Reusable shared components
- Services for API communication
- RxJS for asynchronous streams
- WebSocket service for real-time events
- Route guards
- HTTP interceptor

These are the baseline frontend architecture rules.

---

# 7. Angular Project Structure

Recommended:

```text
aura-client/
│
├── src/
│   ├── app/
│   │
│   ├── core/
│   │   ├── auth/
│   │   ├── guards/
│   │   ├── interceptors/
│   │   ├── services/
│   │   └── models/
│   │
│   ├── shared/
│   │   ├── components/
│   │   ├── directives/
│   │   ├── pipes/
│   │   └── models/
│   │
│   ├── features/
│   │   ├── dashboard/
│   │   ├── conversations/
│   │   ├── tasks/
│   │   ├── knowledge/
│   │   ├── interactions/
│   │   ├── representative/
│   │   ├── settings/
│   │   └── auth/
│   │
│   ├── layout/
│   │   ├── shell/
│   │   ├── sidebar/
│   │   └── header/
│   │
│   └── app.routes.ts
│
└── ...
```

---

# 8. Angular Core Layer

`core/` contains application-wide infrastructure.

Examples:

```text
AuthService
ApiErrorService
SignalRService
TokenService
AppConfigService
```

Core services should generally have application-wide responsibility.

Do not put feature-specific business logic into `core`.

---

# 9. Angular Shared Layer

`shared/` contains reusable UI functionality.

Examples:

```text
LoadingSpinner
ConfirmationDialog
EmptyState
ErrorMessage
VoiceButton
StatusIndicator
```

Shared components should not depend on specific feature implementations.

Incorrect:

```text
SharedButton
 ↓
TaskService
```

Correct:

```text
SharedButton
 ↓
Generic input/output
```

---

# 10. Angular Feature Layer

Each major product capability should have its own feature area.

Example:

```text
features/
   conversations/
       components/
       services/
       models/
       pages/

   tasks/
       components/
       services/
       models/
       pages/
```

Feature-specific logic belongs inside the feature unless it is genuinely reusable.

---

# 11. Angular API Communication

Components must not directly contain HTTP implementation.

Correct:

```text
ConversationComponent
        ↓
ConversationService
        ↓
HttpClient
        ↓
.NET API
```

Incorrect:

```text
ConversationComponent
        ↓
HttpClient
        ↓
.NET API
```

The API contract is defined in:

```text
/docs/API.md
```

---

# 12. Angular Real-Time Communication

Use:

```text
SignalRService
```

for SignalR connection management.

Architecture:

```text
SignalR
   ↓
SignalRService
   ↓
RxJS Observable
   ↓
Feature Service / Component
   ↓
UI State
```

Components should not independently create multiple SignalR connections.

---

# 13. Angular State Strategy

Do not introduce a large state-management library initially.

Use:

```text
Component state
+
Angular services
+
RxJS
```

Introduce a dedicated state-management library only if actual complexity requires it.

The goal is to understand and demonstrate RxJS without adding unnecessary infrastructure.

---

# 14. Angular HTTP Interceptor

A global HTTP interceptor should eventually handle:

```text
JWT attachment
API error handling
Common HTTP behavior
```

Components should not manually add:

```text
Authorization: Bearer ...
```

---

# 15. Angular Route Guards

Protected pages should use route guards.

Example:

```text
/login
    ↓
public

/dashboard
/conversations
/tasks
/knowledge
/interactions
/settings
    ↓
authenticated
```

The exact route configuration will be implemented during authentication.

---

# 16. Backend Architecture

.NET 8 Web API is the main backend.

Recommended project structure:

```text
aura-api/
│
├── AURA.API/
│   ├── Controllers/
│   ├── Hubs/
│   ├── Middleware/
│   ├── Extensions/
│   └── Program.cs
│
├── AURA.Application/
│   ├── Interfaces/
│   ├── Services/
│   ├── DTOs/
│   ├── Validators/
│   ├── UseCases/
│   ├── Agent/
│   ├── RAG/
│   ├── Voice/
│   └── Workflows/
│
├── AURA.Domain/
│   ├── Entities/
│   ├── Enums/
│   ├── ValueObjects/
│   └── Rules/
│
└── AURA.Infrastructure/
    ├── Persistence/
    ├── AI/
    ├── VectorStore/
    ├── STT/
    ├── TTS/
    ├── Workflows/
    └── ExternalServices/
```

---

# 17. API Layer

`AURA.API` is responsible for:

```text
HTTP requests
HTTP responses
Authentication middleware
Authorization
Routing
SignalR hubs
Global middleware
Dependency injection configuration
```

Controllers should remain thin.

Example:

```text
ConversationController
       ↓
ConversationService
       ↓
AgentOrchestrator
```

Not:

```text
ConversationController
       ↓
Ollama
       ↓
MySQL
       ↓
Qdrant
```

---

# 18. Application Layer

The Application layer contains use cases and application orchestration.

Examples:

```text
CreateConversation
SendMessage
CreateTask
GetTasks
UploadKnowledgeDocument
SearchKnowledge
CompleteInteraction
GenerateInteractionReport
```

This layer coordinates business operations.

---

# 19. Domain Layer

The Domain layer contains core business concepts.

Examples:

```text
User
Conversation
Message
Task
Interaction
InteractionMessage
ActionItem
KnowledgeDocument
KnowledgeChunk
AgentMemory
```

Domain logic should not depend on:

```text
Angular
MySQL
Qdrant
Ollama
n8n
HTTP
```

---

# 20. Infrastructure Layer

Infrastructure implements external dependencies.

Examples:

```text
MySQL repositories
Ollama client
BGE-M3 embedding implementation
Qdrant client
faster-whisper integration
Piper integration
n8n client
file storage
```

This keeps external providers replaceable.

---

# 21. Dependency Direction

The intended dependency direction is:

```text
AURA.API
    ↓
AURA.Application
    ↓
AURA.Domain

AURA.Infrastructure
    ↓
AURA.Application
    ↓
AURA.Domain
```

The Domain layer should remain independent.

Conceptually:

```text
          API
           |
           v
     Application
       /      \
      v        v
  Domain   Interfaces
             ^
             |
       Infrastructure
```

---

# 22. AI Provider Abstraction

The backend must not make the entire application dependent on one AI provider.

Define interfaces such as:

```text
IAiChatService
IAiEmbeddingService
ISttService
ITtsService
IVectorStore
IN8nService
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
n8n
```

The selected stack follows the project AI Stack specification.

---

# 23. Agent Architecture

AURA's agent is not simply:

```text
User
 ↓
LLM
 ↓
Answer
```

It is:

```text
User Message
      ↓
Conversation Service
      ↓
Agent Orchestrator
      ↓
Context Builder
      ↓
LLM
      ↓
Decision
  ┌───┼────┐
  │   │    │
  ▼   ▼    ▼
Answer RAG Tool
  │   │    │
  │   ▼    ▼
  │ Qdrant Tool Registry
  │        │
  │        ▼
  │   Application Service
  │        │
  │       DB
  │
  └────┬───┘
       ↓
Final LLM response
       ↓
Conversation persistence
```

---

# 24. Agent Orchestrator

The Agent Orchestrator is responsible for coordinating:

```text
Conversation context
LLM interaction
RAG
Tool calling
Tool results
Response generation
Confirmation handling
Agent state
```

It should NOT directly contain:

```text
SQL queries
Angular code
UI logic
Qdrant implementation details
Ollama-specific application logic
```

Those belong behind appropriate abstractions.

---

# 25. Tool Registry

AURA initially supports:

```text
CreateTask
GetTasks
UpdateTask
CompleteTask
GetInteraction
```

Architecture:

```text
LLM
 ↓
Tool Request
 ↓
Tool Registry
 ↓
Tool Definition
 ↓
Authorization
 ↓
Application Service
 ↓
Repository
 ↓
MySQL
```

New tools should be addable without rewriting the core agent.

This directly supports the BRS requirement that new AI tools should be addable without rewriting the core agent.

---

# 26. Tool Definition

Every tool should define:

```text
Name
Description
Input schema
Authorization requirement
Confirmation requirement
Execution handler
Result schema
```

Example:

```text
CreateTask
```

Conceptual definition:

```text
Name:
CreateTask

Purpose:
Create a task for the authenticated user.

Input:
title
description
priority
dueDate

Authorization:
Authenticated user

Confirmation:
Based on configured action policy

Execution:
TaskApplicationService

Result:
Created task identifier/status
```

---

# 27. Tool Security

The LLM is NOT trusted.

A tool request must be treated as untrusted input.

Flow:

```text
LLM
 ↓
Tool request
 ↓
Schema validation
 ↓
Authorization
 ↓
Business validation
 ↓
Confirmation if required
 ↓
Execute
```

Never:

```text
LLM
 ↓
SQL
```

---

# 28. RAG Architecture

RAG consists of:

```text
Document ingestion
        ↓
Text extraction
        ↓
Chunking
        ↓
Embeddings
        ↓
Qdrant
        ↓
Semantic retrieval
        ↓
Context
        ↓
LLM
        ↓
Grounded answer
```

The BRS explicitly requires document extraction, chunking, embeddings, vector storage, semantic retrieval, and grounded generation.

---

# 29. RAG Storage Architecture

```text
                  Knowledge Document
                         |
                         v
                      MySQL
                         |
                    Document ID
                         |
                         v
                   Text chunks
                    /       \
                   /         \
                  v           v
              MySQL        BGE-M3
              metadata     embedding
                              |
                              v
                            Qdrant
```

MySQL:

```text
Document metadata
Chunk metadata
Ownership
Processing status
```

Qdrant:

```text
Vector
Vector metadata
Semantic search
```

---

# 30. RAG Security

Every vector retrieval must enforce ownership.

Conceptually:

```text
Current User
     ↓
UserId
     ↓
Qdrant filter
     ↓
Allowed documents
     ↓
Top-K chunks
```

Never perform unrestricted vector search across all users.

The BRS explicitly requires RAG retrieval to enforce user/document ownership.

---

# 31. Voice Architecture

Initial voice architecture is intentionally simple.

```text
Angular Microphone
       ↓
Audio capture
       ↓
.NET Voice API
       ↓
faster-whisper
       ↓
Text
       ↓
Agent
       ↓
Text Response
       ↓
Piper
       ↓
Audio
       ↓
Angular
       ↓
Speaker
```

Do not begin with full-duplex streaming.

---

# 32. Voice Real-Time Architecture

SignalR provides processing-state updates.

Example:

```text
Microphone
    ↓
voice.listening
    ↓
voice.transcribing
    ↓
agent.processing
    ↓
rag.searching
    ↓
tool.started
    ↓
tool.completed
    ↓
agent.responding
    ↓
voice.speaking
    ↓
conversation.completed
```

These events are defined in the API communication contract.

---

# 33. SignalR Architecture

SignalR is used for application events, not as the main CRUD transport.

```text
HTTP
 ↓
Commands / Queries

SignalR
 ↓
Real-time state/events
```

SignalR Hub:

```text
AURAHub
```

The exact hub path/name may be finalized during implementation.

---

# 34. SignalR Connection Flow

```text
Angular
   ↓
SignalRService
   ↓
Connect to AURA Hub
   ↓
Subscribe to events
   ↓
Receive events
   ↓
Update RxJS state
   ↓
UI
```

The frontend should establish one appropriate connection rather than opening independent connections per component.

---

# 35. Interaction Architecture

Representative Mode introduces a second interaction type.

```text
Primary User
      |
      v
AURA

External Participant
      |
      v
Representative Interface
      |
      v
Interaction
      |
      v
Agent
      |
  +---+---+
  |       |
 RAG     Tools
  |       |
  +---+---+
      |
      v
Transcript
      |
      v
Interaction Complete
```

For MVP, the external participant is simulated through a web interface.

Real phone integration is future scope.

---

# 36. Representative Mode

Representative Mode must operate under configured boundaries.

The agent must not automatically assume that it can:

```text
Make commitments
Approve important decisions
Expose private information
Perform unrestricted actions
Represent the user's opinions without configured permission
```

The BRS requires representation rules to control what AURA can say/do.

---

# 37. Interaction Processing

When an interaction completes:

```text
Interaction
      ↓
Persist final transcript
      ↓
Complete interaction
      ↓
Trigger workflow
      ↓
n8n
      ↓
Summary
      ↓
Action Items
      ↓
Optional Task
      ↓
Notification
```

The source interaction remains persisted even if automation fails.

---

# 38. n8n Architecture

n8n is an automation system, not AURA's primary business-data store.

```text
.NET Backend
     |
     | Webhook
     v
    n8n
     |
     +---- Summary
     |
     +---- Action Items
     |
     +---- Task creation
     |
     +---- Notification
```

The BRS requires backend-to-n8n integration, completed-interaction processing, task creation, and notification workflows.

---

# 39. n8n Failure Handling

If n8n is unavailable:

```text
Interaction
     ↓
MySQL
     ↓
Saved successfully
     ↓
n8n unavailable
     ↓
Log failure / retry strategy
```

Do NOT:

```text
n8n unavailable
     ↓
Delete interaction
```

The BRS explicitly states that automation failure must not destroy the interaction record.

---

# 40. MySQL Architecture

MySQL is the relational system of record.

It stores:

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

See:

```text
/docs/DATABASE.md
```

for detailed schema.

---

# 41. Qdrant Architecture

Qdrant is responsible for:

```text
Vector storage
Semantic similarity search
Metadata filtering
RAG retrieval
```

It should not replace MySQL.

Correct:

```text
MySQL = relational truth

Qdrant = vector search
```

---

# 42. File Storage

Knowledge files require a storage abstraction.

The application should not permanently depend on:

```text
C:\AURA\files\
```

Use an abstraction such as:

```text
IFileStorageService
```

Possible implementations:

```text
LocalFileStorage
ObjectStorage
```

Initial development may use local storage.

---

# 43. Authentication Architecture

Authentication uses:

```text
ASP.NET Core Identity
+
JWT
```

Flow:

```text
Angular
   ↓
Login
   ↓
.NET Authentication
   ↓
JWT
   ↓
Angular stores/handles token appropriately
   ↓
HTTP Interceptor
   ↓
Authorization header
   ↓
Protected API
```

Protected APIs must require authentication.

The BRS explicitly requires protected APIs and user-data isolation.

---

# 44. Authorization Architecture

Authorization is enforced in the backend.

Example:

```text
GET /api/tasks/123
        ↓
Authenticate
        ↓
Get Current User
        ↓
Find Task 123
        ↓
Verify Task.UserId == CurrentUser.Id
        ↓
Allow / Deny
```

The frontend is not a security boundary.

---

# 45. User Data Isolation

Every user-owned resource must be isolated.

Examples:

```text
User A
 ├── Conversations
 ├── Tasks
 ├── Knowledge
 ├── Interactions
 └── Memories

User B
 ├── Conversations
 ├── Tasks
 ├── Knowledge
 ├── Interactions
 └── Memories
```

No user should be able to access another user's data.

---

# 46. Configuration

Configuration must not be hardcoded.

Examples:

```text
Database connection
Ollama URL
Qdrant URL
n8n URL
JWT settings
Storage path
AI model name
```

Use:

```text
appsettings.json
environment-specific configuration
environment variables
user secrets where appropriate
```

Secrets must never be committed.

---

# 47. External Provider Configuration

Example conceptual configuration:

```text
AI:
  Provider: Ollama
  Model: qwen3:8b

Embedding:
  Provider: Local
  Model: BGE-M3

VectorStore:
  Provider: Qdrant

STT:
  Provider: FasterWhisper

TTS:
  Provider: Piper

Workflow:
  Provider: n8n
```

The exact configuration structure will be finalized during implementation.

---

# 48. Error Handling Architecture

Errors should flow through consistent layers.

```text
Infrastructure Error
        ↓
Application handling
        ↓
Meaningful application exception
        ↓
API middleware
        ↓
HTTP error response
        ↓
Angular error handler
        ↓
User-friendly message
```

Do not expose:

```text
Stack traces
SQL queries
File paths containing secrets
API keys
Internal infrastructure details
```

---

# 49. Observability

Important operations should be logged.

Examples:

```text
API request failures
AI service failures
Tool execution
RAG failures
Document processing failures
n8n failures
Authentication failures
Unexpected exceptions
```

The BRS requires important operations to be logged and AI actions to be auditable.

Do not log sensitive content unnecessarily.

---

# 50. AI Action Audit

AI actions should be traceable.

Example:

```text
Conversation
    ↓
Message
    ↓
Agent decision
    ↓
Tool
    ↓
Execution
    ↓
Result
```

Potential audit storage is described in `DATABASE.md`.

The exact audit implementation will be introduced when the security/observability phase is implemented.

---

# 51. Request Correlation

Important backend operations should have a correlation/trace identifier.

Conceptually:

```text
Request
   ↓
TraceId = abc123
   ↓
Controller
   ↓
Agent
   ↓
RAG
   ↓
Tool
   ↓
Database
```

This allows failures to be traced across multiple services/components.

---

# 52. Async Processing

Use asynchronous programming for operations that can block or take significant time.

Examples:

```text
Database access
LLM requests
Embedding generation
Vector search
STT
TTS
File processing
n8n calls
```

Avoid unnecessary synchronous blocking.

---

# 53. Background Processing

Some operations may eventually use background processing.

Candidates:

```text
Document processing
Embedding generation
Interaction report generation
Notification processing
Retry workflows
```

Do not introduce a complex distributed job system initially.

Use the simplest reliable mechanism that satisfies the requirement.

---

# 54. Docker Architecture

Docker Compose should initially be used selectively.

Recommended containerized infrastructure:

```text
docker-compose.yml

services:
  mysql
  qdrant
  n8n
```

Ollama may initially run natively to simplify local GPU/hardware access.

Angular and .NET can initially run directly during development.

---

# 55. Local Development Architecture

Target local environment:

```text
Browser
   |
Angular Dev Server
   |
.NET API
   |
   +---- MySQL
   |
   +---- Ollama
   |
   +---- Qdrant
   |
   +---- faster-whisper
   |
   +---- Piper
   |
   +---- n8n
```

The exact process/container split may change based on hardware and development convenience.

---

# 56. Dependency Boundaries

The following direct dependencies are prohibited:

```text
Angular → MySQL
Angular → Qdrant
Angular → Ollama
Angular → n8n

LLM → MySQL
LLM → Qdrant directly

Controller → SQL
Controller → Ollama directly
Controller → Qdrant directly
```

Correct:

```text
Angular
   ↓
.NET API
   ↓
Application
   ↓
Interfaces
   ↓
Infrastructure
```

---

# 57. Complete Text Conversation Flow

```text
Angular
   |
   | POST message
   v
ConversationController
   |
   v
ConversationApplicationService
   |
   +---- Persist User Message
   |
   v
AgentOrchestrator
   |
   +---- Build Context
   |
   v
LLM
   |
   +---- Direct Answer
   |
   +---- RAG
   |
   +---- Tool
   |
   v
Final Response
   |
   +---- Persist Assistant Message
   |
   v
API Response
   |
   v
Angular
```

---

# 58. Complete RAG Flow

```text
User Question
      ↓
Conversation API
      ↓
Agent Orchestrator
      ↓
LLM decides knowledge is required
      ↓
Embedding Service
      ↓
BGE-M3
      ↓
Qdrant
      ↓
User/document filtered search
      ↓
Relevant chunks
      ↓
Context Builder
      ↓
LLM
      ↓
Grounded answer
      ↓
Conversation
```

---

# 59. Complete Tool Flow

```text
User:
"Create a task to review UCC validation."

        ↓

Conversation API

        ↓

Agent Orchestrator

        ↓

LLM

        ↓

CreateTask Tool Request

        ↓

Tool Registry

        ↓

Validate arguments

        ↓

Authorize user

        ↓

Confirmation if required

        ↓

Task Application Service

        ↓

Task Repository

        ↓

MySQL

        ↓

Tool Result

        ↓

LLM

        ↓

Assistant Response
```

---

# 60. Complete Voice Flow

```text
User speaks
      ↓
Angular microphone
      ↓
Audio capture
      ↓
STT API
      ↓
faster-whisper
      ↓
Text
      ↓
Conversation API
      ↓
Agent
      ↓
Response text
      ↓
TTS API
      ↓
Piper
      ↓
Audio
      ↓
Angular
      ↓
Speaker
```

---

# 61. Complete Representative Flow

```text
External Participant
       ↓
Representative UI
       ↓
Voice/Text
       ↓
STT if voice
       ↓
Representative Agent
       ↓
Representation Rules
       ↓
RAG / Tools if authorized
       ↓
Response
       ↓
Participant
       ↓
Transcript stored
       ↓
Interaction completed
       ↓
n8n
       ↓
Summary
       ↓
Action Items
       ↓
Task
       ↓
Notification
       ↓
Owner reviews report
```

This represents the core product scenario described by the BRS.

---

# 62. Module Boundaries

Initial backend modules:

```text
Identity
Conversations
Tasks
Knowledge
Agent
Voice
Interactions
Memory
Workflows
Notifications
```

These are logical modules inside the modular monolith.

They are NOT separate microservices.

---

# 63. Module Communication

Modules should communicate through application interfaces/use cases rather than directly manipulating each other's database tables.

Example:

```text
Agent
 ↓
Task Application Service
 ↓
Task Repository
```

Not:

```text
Agent
 ↓
Task table SQL
```

---

# 64. Conversation Module

Owns:

```text
Conversation
Message
Conversation lifecycle
Conversation history
Message persistence
```

It may invoke:

```text
Agent module
```

but should not implement LLM reasoning itself.

---

# 65. Task Module

Owns:

```text
Task creation
Task retrieval
Task update
Task completion
Task deletion
Task validation
```

The Agent module uses the Task module through a tool.

---

# 66. Knowledge Module

Owns:

```text
Document upload
Document metadata
Document processing
Chunking
Embedding orchestration
Vector storage coordination
Knowledge retrieval
Source references
```

The Agent calls the Knowledge/RAG service rather than directly accessing Qdrant.

---

# 67. Interaction Module

Owns:

```text
Interaction lifecycle
Transcript
Action items
Summary
Priority
Interaction history
```

It may trigger workflow processing when the interaction completes.

---

# 68. Voice Module

Owns:

```text
Audio handling
STT
TTS
Voice session state
Voice provider abstraction
```

The Voice module should not implement conversation business logic.

---

# 69. Workflow Module

Owns:

```text
n8n communication
Webhook triggering
Workflow status
Automation error handling
```

It should not become the source of truth for AURA data.

---

# 70. Memory Module

Owns:

```text
Memory creation
Memory retrieval
Memory update
Memory deletion
Memory safety rules
```

Only permitted non-sensitive memory should be stored.

---

# 71. Future Extension Architecture

Future integrations may include:

```text
Phone
WhatsApp
Email
Calendar
AI Interviewer
Meeting Assistant
```

They should connect through new adapters/modules rather than rewriting the agent core.

Example:

```text
Agent
  |
  +-- Tool Registry
        |
        +-- Task Tool
        +-- Calendar Tool
        +-- Email Tool
        +-- WhatsApp Tool
        +-- Phone Tool
```

These are future extensions and must not block MVP.

---

# 72. Why Modular Monolith Instead of Microservices?

AURA is intentionally not being built as microservices initially.

Reasons:

```text
Smaller project
Local-first development
Lower operational complexity
Easier debugging
Faster AI-assisted development
Simpler deployment
Clear enough module boundaries
```

The BRS explicitly warns against introducing microservices and other infrastructure merely to make the project appear more sophisticated.

The architecture can later evolve if actual requirements justify it.

---

# 73. Why Not Kafka?

AURA does not initially require Kafka.

SignalR handles frontend real-time events.

n8n handles workflow automation.

Internal application services handle business orchestration.

Therefore:

```text
No Kafka initially.
```

---

# 74. Why Not Redis?

Redis is not part of the initial architecture.

Do not introduce Redis for:

```text
Simple caching
Session storage
Message passing
AI memory
```

unless an actual requirement appears.

---

# 75. Why Not Multiple Agents?

AURA initially uses one primary agent with tools.

```text
One Agent
   |
   +-- RAG
   +-- Tools
   +-- Memory
   +-- Voice
```

Do not create multiple specialized agents without a real use case.

The BRS explicitly discourages excessive AI agents.

---

# 76. AI Coding Tool Contract

Because AURA will be developed by multiple AI coding tools, every tool must treat the repository documentation as shared project context.

Before coding:

```text
Read:
BRS.md
ARCHITECTURE.md
DATABASE.md
API.md
AI.md
```

Then identify:

```text
Feature
Module
API contract
Database entities
Dependencies
Events
Security rules
Acceptance criteria
```

---

# 77. AI Agent Must Not Assume

An AI coding tool must NOT assume:

```text
"Since this is common practice, I can add it."

"Since another project uses Redis, AURA needs Redis."

"I'll create an endpoint because it is convenient."

"I'll change the response structure."

"I'll add a database column."

"I'll directly call Ollama here."

"I'll query MySQL from the tool."

"I'll create another microservice."
```

Architecture changes require justification.

---

# 78. AI-to-AI Handoff

If one AI tool completes work and another continues:

The first tool should leave:

```text
Working code
Updated documentation
Clear interfaces
Tests
No hidden assumptions
```

The next AI tool should be able to understand the implementation from the repository.

No architectural knowledge should exist only in conversation history.

---

# 79. Human Supervision Model

The development model is:

```text
Human
  |
  | Defines requirement / reviews
  v
AI Coding Tool
  |
  | Generates implementation
  v
Repository
  |
  | Tests / build / review
  v
Human
```

The human is responsible for:

```text
Requirement approval
Architecture approval
Tool approval
Security review
Final code review
Acceptance testing
```

AI tools are responsible for implementation within the approved contract.

---

# 80. Definition of Done

A feature is architecturally complete when:

```text
BRS requirement
      ↓
Architecture
      ↓
API contract
      ↓
Database contract
      ↓
Implementation
      ↓
Tests
      ↓
Error handling
      ↓
Security
      ↓
Logging
      ↓
Documentation
```

Code compiling is not sufficient.

---

# 81. Development Phases

The architecture supports the following implementation order:

```text
Phase 0
Specification
      ↓
Phase 1
Foundation
      ↓
Phase 2
Text Assistant
      ↓
Phase 3
Agent + Tools
      ↓
Phase 4
RAG
      ↓
Phase 5
Voice
      ↓
Phase 6
Representative Mode
      ↓
Phase 7
n8n Automation
      ↓
Phase 8
Security + Polish
```

This follows the BRS implementation order.

---

# 82. Phase 1 Architecture

The first runnable architecture is intentionally small:

```text
Angular 17
     |
     | HTTP
     v
.NET 8 API
     |
     v
MySQL
```

First endpoint:

```text
GET /api/health
```

Expected:

```json
{
  "status": "Healthy",
  "application": "AURA"
}
```

The BRS explicitly defines this as the first implementation milestone.

---

# 83. Phase 2 Architecture

Add:

```text
Angular Chat
      ↓
Conversation API
      ↓
Application
      ↓
Ollama / Qwen3
      ↓
MySQL
```

Acceptance:

```text
User can have a persistent text conversation with AURA.
```

---

# 84. Phase 3 Architecture

Add:

```text
Agent Orchestrator
      ↓
Tool Registry
      ↓
Task Module
      ↓
MySQL
```

Acceptance:

```text
Natural language can perform task operations.
```

---

# 85. Phase 4 Architecture

Add:

```text
Knowledge Module
      ↓
BGE-M3
      ↓
Qdrant
      ↓
Agent
```

Acceptance:

```text
AURA answers questions using uploaded private knowledge.
```

---

# 86. Phase 5 Architecture

Add:

```text
Voice Module
      ↓
faster-whisper
      ↓
Agent
      ↓
Piper

SignalR
      ↓
Real-time UI state
```

Acceptance:

```text
User can speak to AURA and hear the response.
```

---

# 87. Phase 6 Architecture

Add:

```text
Representative Module
      ↓
Interaction
      ↓
Transcript
      ↓
Summary
      ↓
Action Items
```

Acceptance:

```text
Simulated external participant can interact with AURA.
```

---

# 88. Phase 7 Architecture

Add:

```text
Interaction Completed
      ↓
n8n
      ↓
Summary / Action Items / Task / Notification
```

Acceptance:

```text
Completed interaction automatically triggers the workflow.
```

---

# 89. Architecture Guardrails

Do NOT initially add:

```text
Kubernetes
Microservices
Kafka
Complex event buses
Redis
Multiple LLM agents
Dozens of tools
Complex cloud infrastructure
Real telephony
```

unless a real requirement emerges.

The objective is:

> Strong implementation depth > number of technologies.

This is directly aligned with the BRS scope guardrails.

---

# 90. Architecture Decision Summary

```text
Frontend:
Angular 17

Backend:
.NET 8 Web API

Architecture:
Modular Monolith

Backend Style:
Clean Architecture-style separation

Database:
MySQL

Vector Database:
Qdrant

LLM Runtime:
Ollama

Primary LLM:
Qwen3 8B

STT:
faster-whisper

TTS:
Piper

Embedding:
BGE-M3

Automation:
n8n

Real-time:
ASP.NET Core SignalR

Authentication:
ASP.NET Core Identity + JWT

Containerization:
Docker Compose

AI Strategy:
Local-first + provider abstraction
```

This matches the current AURA AI Stack decision.

---

# 91. Final Architecture Principle

> **AURA is a modular monolith where Angular communicates with .NET through explicit API and SignalR contracts. The .NET application owns business logic, authorization, AI orchestration, and tool execution. MySQL is the relational system of record, Qdrant is the vector-search system, AI providers are accessed through abstractions, and n8n is an automation layer rather than a business-data store. Each module has a clear responsibility, and the architecture must remain simple enough for local development while being extensible enough for future representative capabilities.**

---

# 92. AI Coding Agent Final Instruction

Before changing AURA architecture or implementing a feature:

```text
1. Read BRS.md.
2. Read API.md.
3. Read DATABASE.md.
4. Read AI.md.
5. Read ARCHITECTURE.md.
6. Identify the module.
7. Identify the API contract.
8. Identify database dependencies.
9. Identify AI dependencies.
10. Identify SignalR events.
11. Identify security requirements.
12. Implement within existing boundaries.
13. Run tests/build.
14. Update documentation if the contract changed.
```

If an architectural decision is not defined:

```text
Do not silently introduce a major technology.

Identify the gap.
Propose the smallest appropriate solution.
Document the decision.
Then implement.
```

---

# 93. Architecture Status

```text
Architecture style: Defined
Frontend architecture: Defined
Backend architecture: Defined
Module boundaries: Defined
AI boundaries: Defined
RAG boundaries: Defined
Voice boundaries: Defined
Database boundary: Defined
SignalR boundary: Defined
n8n boundary: Defined
Security boundary: Defined
AI coding-agent rules: Defined
Implementation phases: Defined
```

**Architecture baseline v1.0 is now established.**