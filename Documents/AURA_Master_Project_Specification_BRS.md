# AURA — AI Personal Representative
## Master Project Specification / BRS / AI Development Context
**Version:** 1.0  
**Status:** Development Baseline  
**Project Type:** Portfolio / Learning / Enterprise-style AI application  
**Primary Developer:** Umar Shaikh

---

# 1. Project Summary

AURA (AI-powered Unified Representative Assistant) is a voice-first personal AI agent.

AURA is not intended to be a generic chatbot. Its purpose is to act as an intelligent digital representative that can:

1. Converse with the user through text and voice.
2. Answer questions using a private knowledge base.
3. Retrieve information using RAG.
4. Execute authorized tasks through AI tool calling.
5. Maintain conversation and interaction history.
6. Process interactions automatically using n8n.
7. Generate concise interaction summaries and action items.
8. Provide a foundation for representing the user when the user is unavailable.

The MVP must remain small enough to complete quickly. Architecture should be extensible, but optional features must not delay the MVP.

---

# 2. Core Business Problem

Professionals frequently miss calls, requests, questions, tasks, and other interactions while unavailable.

Typical problem:

User unavailable
-> interaction occurs
-> interaction is missed or only partially captured
-> user returns later
-> user manually asks what happened
-> time is spent reconstructing context
-> important actions may be forgotten

AURA addresses this by allowing an AI representative to understand an interaction, capture useful information, answer within authorized boundaries, perform permitted actions, and produce a report for the user.

Core value proposition:

> Don't miss important interactions just because you are unavailable.

---

# 3. Product Vision

AURA should eventually behave like a digital representative of its owner.

Product hierarchy:

PRODUCT
AURA — AI Personal Representative

CORE
Voice-first AI Assistant

CORE CAPABILITIES
- LLM
- Voice
- RAG
- Memory
- Tool Calling
- Interaction Recording
- Workflow Automation

SIGNATURE CAPABILITY
Represent the user when unavailable

OPTIONAL EXTENSIONS
- Real phone calls
- WhatsApp
- Email
- Calendar
- AI Interviewer
- Meeting Assistant
- Additional external tools
- Multi-user support

---

# 4. Project Goals

## 4.1 Primary Goals

- Build a practical AI agent, not a static chatbot.
- Gain hands-on experience with LLMs.
- Implement real RAG.
- Implement AI tool/function calling.
- Implement speech-to-text and text-to-speech.
- Implement real-time voice interaction.
- Integrate n8n for workflow automation.
- Build the system using Angular + .NET + MySQL.
- Demonstrate enterprise software engineering practices.
- Keep AI services free/local wherever reasonably possible.
- Produce a strong portfolio project suitable for interviews for a 2+ years software developer profile.

## 4.2 Learning Goals

The developer should be able to explain:

- LLM request/response lifecycle
- Prompting
- Conversation context
- Embeddings
- Vector search
- Chunking
- RAG
- Hallucination control
- Agent/tool calling
- Function/tool schemas
- Voice pipeline
- WebSocket communication
- AI workflow automation
- AI safety/authorization
- Backend architecture
- Database design
- Error handling
- Observability

---

# 5. Scope Strategy

The project uses three scope levels.

## P0 — MVP / Must Have

- Angular UI
- .NET 8 backend
- MySQL
- Text conversation
- Voice conversation
- STT
- TTS
- LLM
- Conversation history
- Knowledge document upload
- Document processing
- Embeddings
- Vector database
- RAG
- Tool calling
- Task management
- Interaction recording
- AI summaries
- Action-item extraction
- n8n integration

## P1 — Important

- Authentication
- Authorization
- Agent configuration
- Memory
- Notifications
- Better observability
- Confirmation before risky actions

## P2 — Future

- Real telephone integration
- WhatsApp
- Email
- Calendar
- AI interviewer
- Meeting assistant
- Multi-user SaaS
- Advanced analytics

P2 features must not block the MVP.

---

# 6. Actors

## 6.1 Primary User

The owner of AURA.

Can:
- Talk to AURA
- Upload knowledge
- Ask questions
- Create/update/complete tasks
- Review conversations
- Review interactions
- Configure the agent
- Enable/disable representative mode
- Review AI-generated reports

## 6.2 External Participant

A person interacting with AURA while the owner is unavailable.

Examples:
- Client
- Colleague
- Customer
- Vendor
- Friend

For MVP, external interaction can be simulated through a dedicated web interface. Real phone integration is future scope.

---

# 7. Core User Journeys

## Journey A — Normal Voice Assistant

User opens AURA
-> clicks microphone
-> speaks
-> STT converts speech to text
-> backend sends request to agent
-> agent decides whether to answer directly, use RAG, or call a tool
-> LLM generates response
-> TTS converts response to audio
-> user hears response
-> conversation is stored

## Journey B — Knowledge Question

User:
"According to the uploaded UCC BRS, what validation is required?"

Question
-> embedding
-> vector search
-> relevant document chunks
-> LLM receives question + retrieved context
-> grounded answer
-> source information shown to user
-> conversation stored

## Journey C — Task Creation

User:
"Create a task to review UCC validation tomorrow."

User message
-> LLM detects CreateTask intent
-> extracts title/date/priority
-> validates tool parameters
-> optionally asks confirmation
-> .NET executes CreateTask
-> MySQL stores task
-> result returned to LLM
-> natural language response
-> conversation stored

## Journey D — Representative Mode

External participant:
"Is Umar available?"

AURA:
"Umar is currently unavailable. I can take a message or help with your request."

Participant explains issue
-> AURA asks follow-up questions
-> AURA captures topic/details
-> interaction transcript is recorded
-> interaction is completed
-> n8n workflow is triggered
-> summary generated
-> action items extracted
-> task may be created
-> user is notified
-> user later asks "What happened while I was away?"
-> AURA summarizes the interaction

---

# 8. Functional Requirements

## User Management

FR-001 — User registration shall be supported.

FR-002 — Authenticated users shall access protected AURA data.

FR-003 — User profile information shall be stored.

FR-004 — User shall configure agent preferences.

## Conversation

FR-005 — User shall start a conversation.

FR-006 — User shall send text messages.

FR-007 — AURA shall generate an AI response.

FR-008 — Conversation history shall be persisted.

FR-009 — Relevant conversation context shall be available to the agent.

## Voice

FR-010 — User shall provide microphone input.

FR-011 — Speech shall be converted to text.

FR-012 — AURA response shall be converted to speech.

FR-013 — Voice conversation should provide near-real-time interaction.

## RAG

FR-014 — User shall upload supported documents.

FR-015 — System shall extract document text.

FR-016 — System shall chunk document text.

FR-017 — System shall generate embeddings.

FR-018 — Embeddings shall be stored in a vector database.

FR-019 — Semantic search shall retrieve relevant chunks.

FR-020 — LLM shall answer using retrieved context.

FR-021 — AURA should avoid unsupported claims when answering from private knowledge.

FR-022 — Sources should be displayed for knowledge-based answers.

## Agent

FR-023 — AURA shall identify user intent.

FR-024 — AURA shall select appropriate tools.

FR-025 — AURA shall extract tool parameters.

FR-026 — Backend shall execute authorized tools.

FR-027 — Tool results shall be returned to the agent.

FR-028 — Potentially destructive/important actions should require confirmation.

## Tasks

FR-029 — User shall create tasks.

FR-030 — User shall retrieve tasks.

FR-031 — User shall update tasks.

FR-032 — User shall complete tasks.

## Interactions

FR-033 — Interactions shall be recorded.

FR-034 — Transcript shall be stored where appropriate.

FR-035 — Interaction summary shall be generated.

FR-036 — Action items shall be extracted.

FR-037 — Interaction priority should be determined.

FR-038 — Interaction history shall be viewable.

## Representative

FR-039 — User shall configure availability status.

FR-040 — AURA shall support external interaction in Representative Mode.

FR-041 — Representation rules shall control what AURA can say/do.

FR-042 — AURA shall capture unresolved participant requests.

FR-043 — AURA shall generate an interaction report.

## n8n

FR-044 — Backend shall trigger n8n workflows.

FR-045 — Completed interactions shall be processable by n8n.

FR-046 — n8n should be capable of creating tasks from action items.

FR-047 — n8n should support notification workflows.

## Memory

FR-048 — Conversation context shall be maintained.

FR-049 — Non-sensitive user preferences may be stored as memory.

FR-050 — User should be able to manage stored memory.

## Security

FR-051 — User data shall be isolated by user identity.

FR-052 — Only authorized tools shall execute actions.

FR-053 — RAG retrieval shall enforce user/document ownership.

FR-054 — Protected APIs shall require authentication.

---

# 9. Non-Functional Requirements

NFR-001 — System should provide acceptable conversational latency.

NFR-002 — External AI/service failure shall be handled gracefully.

NFR-003 — API/database/AI failures shall have meaningful error handling.

NFR-004 — Architecture should support future growth.

NFR-005 — Backend shall be modular and maintainable.

NFR-006 — Important operations shall be logged.

NFR-007 — Secrets/API keys shall never be hardcoded.

NFR-008 — New AI tools should be addable without rewriting the core agent.

NFR-009 — AI actions shall be auditable.

NFR-010 — User should be able to understand when an AI action is being performed.

---

# 10. Ten Concrete Use Cases

## UC-001 — Voice Conversation

Actor: Primary User

User speaks a question.

System:
Voice -> STT -> Agent -> LLM/tool/RAG -> response -> TTS -> voice.

Expected result:
User hears an appropriate answer.

## UC-002 — Ask Knowledge Base Question

Actor: Primary User

User asks about uploaded documentation.

System retrieves relevant chunks and generates a grounded response with source reference.

## UC-003 — Upload Knowledge

Actor: Primary User

User uploads PDF/TXT/Markdown.

System extracts -> chunks -> embeds -> stores vectors.

Expected result:
Document is searchable.

## UC-004 — Create Task Through Natural Language

User:
"Create a task to review the UCC validation tomorrow."

Agent calls CreateTask.

Expected result:
Task exists in MySQL.

## UC-005 — Retrieve and Summarize Tasks

User:
"What do I need to do today?"

Agent calls GetTasks and summarizes the result.

## UC-006 — Handle Representative Interaction

External participant interacts with AURA while owner is unavailable.

AURA answers within configured boundaries or captures the request.

Expected result:
Interaction record is created.

## UC-007 — Generate Interaction Report

User:
"What happened while I was away?"

AURA retrieves interaction history and gives a concise summary including participant, topic, priority, and action items.

## UC-008 — Automated Interaction Processing

Interaction completed
-> n8n webhook
-> summary
-> action items
-> task creation if applicable
-> notification.

## UC-009 — Multi-Step Agent

User:
"What tasks are due tomorrow and create a follow-up task for the highest priority one."

Agent:
GetTasks -> analyze -> CreateTask.

Expected result:
Correct task is created after any required confirmation.

## UC-010 — AI Interviewer

Future/P2.

User starts mock interview.

AURA asks dynamic technical questions and generates final feedback.

This must not delay MVP.

---

# 11. Angular UI Specification

The UI should be clean, modern, professional, and AI-product-like without excessive animation.

Primary application layout:

```text
+--------------------------------------------------------------+
| AURA                                     User / Settings     |
+----------------+---------------------------------------------+
| Dashboard      |                                             |
| Assistant      |              Main Content                    |
| Knowledge      |                                             |
| Tasks          |                                             |
| Interactions   |                                             |
| Workflows      |                                             |
| Settings       |                                             |
+----------------+---------------------------------------------+
```

## 11.1 Navigation Menu

MVP should use approximately 7 primary menus:

1. Dashboard
2. Assistant
3. Knowledge
4. Tasks
5. Interactions
6. Workflows
7. Settings

Do not create separate menus for every tiny feature.

### Dashboard

Purpose:
Give the user a quick overview.

Show:
- Greeting
- Agent availability status
- Start conversation button
- Recent conversations
- Pending tasks
- Recent interactions
- Important/high-priority interactions
- Knowledge document count
- Quick actions

Suggested layout:

```text
----------------------------------------------------
Good morning, Umar

[ Start Voice Conversation ]

+----------------+  +----------------+
| Pending Tasks  |  | Interactions   |
|      4         |  |      2         |
+----------------+  +----------------+

Recent Activity
-----------------------------------------------
Time     Type             Summary
10:30    Interaction      API issue reported
09:15    Task             Review BRS
-----------------------------------------------
```

### Assistant

This is the main AI screen.

Desktop layout:

```text
+------------------------------------------------------+
| AURA Assistant                         Voice: Ready  |
+------------------------------------------------------+
|                                                      |
|                  Conversation                        |
|                                                      |
|  User: What tasks do I have today?                  |
|                                                      |
|  AURA: You have 3 tasks...                          |
|                                                      |
|------------------------------------------------------|
| [Type a message........................] [Send]      |
|                                                      |
|                    [  🎙  ]                          |
|                 Hold / Click to talk                |
+------------------------------------------------------+
```

Show:
- Conversation messages
- User/AI distinction
- Voice state
- Microphone button
- Text fallback input
- Stop/cancel button while speaking
- Loading/processing state
- Tool execution indicator
- RAG source indicator where relevant

Voice states:

```text
Idle
Listening
Processing
Speaking
Error
```

Example:

```text
Listening...
"Create a task for tomorrow."
```

During tool execution:

```text
AURA is creating your task...
```

For RAG:

```text
Searching your knowledge base...
```

Avoid exposing raw chain-of-thought. Show only safe high-level status.

### Knowledge

Purpose:
Manage private knowledge.

Views:
- Document list
- Upload area
- Processing status
- Search/test question
- Document details

Layout:

```text
Knowledge Base

[ Upload Document ]

--------------------------------------------------
Document          Type     Status       Uploaded
UCC_BRS.pdf       PDF      Ready        Today
API_Notes.md      MD       Ready        Yesterday
--------------------------------------------------

[Ask your knowledge base]
[........................................] [Ask]
```

Document statuses:

```text
Uploading
Processing
Ready
Failed
```

### Tasks

Show:
- Task title
- Description
- Priority
- Due date
- Status
- Created time

Filters:
- All
- Pending
- Completed
- High Priority

Support:
- Create
- Edit
- Complete
- Delete
- AI creation indicator

### Interactions

Purpose:
Show conversations handled by AURA.

List:

```text
-----------------------------------------------------------
Participant   Topic             Priority   Date
Ahmed         API issue         High       Today
Rahul         Meeting request   Medium     Yesterday
-----------------------------------------------------------
```

Clicking opens detail:

```text
Interaction Details

Participant: Ahmed
Date: ...
Duration: ...
Topic: Production API Issue
Priority: High

Summary
------------------------------------
Production API is returning HTTP 500.

Action Items
------------------------------------
[ ] Investigate API
[ ] Contact Ahmed

Transcript
------------------------------------
...
```

### Workflows

Purpose:
Show automation integration.

Show:
- n8n connection status
- Workflow name
- Trigger
- Last execution
- Status
- Execution result

Example:

```text
Interaction Completed -> Process Interaction
Task Created          -> Task Notification

Status: Connected
```

MVP may provide read-only workflow visibility; actual complex workflow editing stays in n8n.

### Settings

Sections:
- Profile
- Agent configuration
- Voice preferences
- Availability
- Representative rules
- Knowledge settings
- Connected services
- Security

Representative settings:

```text
Availability:
( ) Available
( ) Unavailable
( ) Representative Mode

Allowed actions:
[x] Capture messages
[x] Create tasks
[ ] Send external messages
[ ] Delete data
```

---

# 12. Angular Technical Architecture

Use Angular 17.

Suggested structure:

```text
src/app/
|
+-- core/
|   +-- auth/
|   +-- interceptors/
|   +-- guards/
|   +-- services/
|   +-- models/
|
+-- shared/
|   +-- components/
|   +-- pipes/
|   +-- directives/
|
+-- features/
|   +-- dashboard/
|   +-- assistant/
|   +-- knowledge/
|   +-- tasks/
|   +-- interactions/
|   +-- workflows/
|   +-- settings/
|
+-- layout/
|   +-- sidebar/
|   +-- header/
|
+-- app.routes.ts
```

Principles:
- Feature-based architecture
- Reusable shared components
- Services for API communication
- RxJS for asynchronous streams
- WebSocket service for real-time events
- Route guards for protected pages
- HTTP interceptor for authentication/error handling

---

# 13. .NET Backend Architecture

Use .NET 8 Web API.

Recommended Clean Architecture-style separation:

```text
AURA.API
|
+-- Controllers
+-- Middleware
+-- DependencyInjection
|
AURA.Application
|
+-- Interfaces
+-- Services
+-- DTOs
+-- UseCases
+-- Validators
|
AURA.Domain
|
+-- Entities
+-- Enums
+-- ValueObjects
+-- Domain Rules
|
AURA.Infrastructure
|
+-- Persistence
+-- MySQL
+-- AI
+-- VectorStore
+-- STT
+-- TTS
+-- n8n
+-- External Services
```

Do not over-engineer. The separation exists to keep AI/external providers replaceable.

---

# 14. Backend API Areas

Suggested endpoints:

## Conversation

POST /api/conversations
GET /api/conversations
GET /api/conversations/{id}
POST /api/conversations/{id}/messages

## Voice

POST /api/voice/transcribe
POST /api/voice/synthesize

If real-time provider supports streaming, use WebSocket separately.

## Knowledge

POST /api/knowledge/documents
GET /api/knowledge/documents
GET /api/knowledge/documents/{id}
DELETE /api/knowledge/documents/{id}
POST /api/knowledge/query

## Tasks

POST /api/tasks
GET /api/tasks
GET /api/tasks/{id}
PUT /api/tasks/{id}
DELETE /api/tasks/{id}

## Interactions

GET /api/interactions
GET /api/interactions/{id}
POST /api/interactions
POST /api/interactions/{id}/complete

## Agent

POST /api/agent/chat
POST /api/agent/voice

## Configuration

GET /api/settings
PUT /api/settings

## Workflow

POST /api/workflows/interaction-completed

Actual API names may be adjusted during implementation.

---

# 15. MySQL Database Design

Initial relational model:

## Users

```text
Users
-----
Id PK
Name
Email
Password/AuthReference
AgentName
AvailabilityStatus
CreatedAt
UpdatedAt
```

## Conversations

```text
Conversations
-------------
Id PK
UserId FK
Title
StartedAt
EndedAt
Status
CreatedAt
```

## Messages

```text
Messages
--------
Id PK
ConversationId FK
Role
Content
MessageType
CreatedAt
```

Role:
- User
- Assistant
- System
- Tool

MessageType:
- Text
- Voice
- ToolResult
- RAGResponse

## Tasks

```text
Tasks
-----
Id PK
UserId FK
Title
Description
Priority
DueDate
Status
CreatedAt
UpdatedAt
```

## Interactions

```text
Interactions
------------
Id PK
UserId FK
ParticipantName
ParticipantContact
Topic
Priority
Summary
StartedAt
EndedAt
Status
CreatedAt
```

## InteractionMessages

```text
InteractionMessages
-------------------
Id PK
InteractionId FK
Role
Content
CreatedAt
```

## ActionItems

```text
ActionItems
-----------
Id PK
InteractionId FK
TaskId FK nullable
Description
Priority
Status
CreatedAt
```

## KnowledgeDocuments

```text
KnowledgeDocuments
------------------
Id PK
UserId FK
FileName
FileType
FilePath/StorageReference
Status
CreatedAt
ProcessedAt
```

## KnowledgeChunks

```text
KnowledgeChunks
---------------
Id PK
DocumentId FK
ChunkIndex
Content
VectorReference
MetadataJson
CreatedAt
```

Vector data may be stored in a dedicated vector database rather than MySQL.

## AgentMemories

```text
AgentMemories
-------------
Id PK
UserId FK
MemoryType
Content
Importance
CreatedAt
UpdatedAt
```

Memory implementation can initially be minimal.

---

# 16. Database Relationships

```text
User
 |
 +---- Conversations
 |          |
 |          +---- Messages
 |
 +---- Tasks
 |
 +---- Interactions
 |          |
 |          +---- InteractionMessages
 |          |
 |          +---- ActionItems
 |
 +---- KnowledgeDocuments
 |          |
 |          +---- KnowledgeChunks
 |
 +---- AgentMemories
```

All user-owned records must be filtered by UserId.

---

# 17. LLM Architecture

AURA should use an abstraction around the LLM.

Do not hardcode the entire application to one provider.

Conceptually:

```text
IAiChatService
IAiEmbeddingService
IAiToolService
```

Possible implementations:

```text
OpenAI-compatible API
Local LLM
Other provider
```

The exact provider is a configuration decision.

LLM responsibilities:

- Understand user input
- Maintain conversational context
- Determine whether RAG is needed
- Determine whether a tool is needed
- Select tools
- Extract parameters
- Generate final response
- Summarize interactions
- Extract action items

The LLM should NOT directly access MySQL.

Instead:

```text
LLM
 |
Tool request
 |
.NET Agent Service
 |
Authorized Tool
 |
Repository/Service
 |
MySQL
```

---

# 18. Agent Architecture

Agent flow:

```text
User Input
    |
    v
Conversation Service
    |
    v
Agent Orchestrator
    |
    +---- Need RAG? ------> RAG Service
    |
    +---- Need Tool? -----> Tool Registry
    |
    +---- Normal answer --> LLM
    |
    v
Final Response
```

Agent loop:

```text
Receive input
   |
Build context
   |
Send to LLM with tool definitions
   |
LLM chooses:
   |
   +-- final answer
   |
   +-- tool call
           |
           v
      validate
           |
           v
      authorization
           |
           v
      execute tool
           |
           v
      return result
           |
           v
      LLM generates final response
```

Maximum tool-call/iteration limits should exist to prevent loops.

---

# 19. Tool Calling

Initial tools:

1. CreateTask
2. GetTasks
3. UpdateTask
4. CompleteTask
5. GetInteraction

Optional later:
- CreateNote
- SearchKnowledge
- CreateCalendarEvent
- SendNotification

Tool definition concept:

```text
CreateTask
Description:
Create a new task for the current user.

Parameters:
title
description
priority
dueDate
```

Tool execution must:
- Validate parameters
- Verify current user
- Apply authorization
- Execute service method
- Return structured result
- Log execution

---

# 20. RAG Architecture

RAG pipeline:

```text
                 DOCUMENT INGESTION

PDF/TXT/MD
    |
    v
Text Extraction
    |
    v
Cleaning
    |
    v
Chunking
    |
    v
Embedding Model
    |
    v
Vector Database
```

Query pipeline:

```text
User Question
     |
     v
Question Embedding
     |
     v
Vector Similarity Search
     |
     v
Top-K Chunks
     |
     v
Optional metadata filtering
     |
     v
Prompt + Retrieved Context
     |
     v
LLM
     |
     v
Grounded Answer + Sources
```

Important RAG concepts to implement/understand:

- Chunk size
- Chunk overlap
- Embedding dimensions
- Similarity metric
- Top-K retrieval
- Metadata filtering
- Source citation
- Context limits
- Retrieval failure
- Hallucination control

Do not blindly retrieve every document.

---

# 21. RAG Data Isolation

A query must never retrieve another user's documents.

Concept:

```text
Query
 |
 +-- UserId filter
 |
 +-- allowed knowledge scope
 |
 v
Vector Search
```

Metadata should include:

```text
UserId
DocumentId
FileName
ChunkIndex
```

---

# 22. Voice Architecture

Voice pipeline:

```text
                 USER
                  |
             Microphone
                  |
                  v
             Audio Stream
                  |
                  v
                 STT
                  |
                  v
             Text Input
                  |
                  v
              AI Agent
                  |
                  v
             Text Response
                  |
                  v
                 TTS
                  |
                  v
             Audio Stream
                  |
                  v
               Speaker
```

For MVP:
- Browser microphone
- Browser-compatible audio handling
- STT
- AI backend
- TTS
- Playback

Later:
- Streaming STT
- Streaming LLM
- Streaming TTS
- Full duplex voice
- Phone integration

---

# 23. WebSocket Architecture

WebSocket is intended for real-time conversation state/events.

Possible events:

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

Example:

```text
Angular
  |
WebSocket
  |
.NET Conversation Hub/Service
  |
Agent
  |
events
  |
Angular
```

HTTP can remain the default for normal CRUD operations.

---

# 24. n8n Architecture

n8n is the workflow automation layer.

AURA should not put every automation rule inside .NET.

Example:

```text
.NET
 |
Webhook
 |
n8n
 |
+--> Get Interaction
 |
+--> AI Summary
 |
+--> Extract Action Items
 |
+--> Create Task
 |
+--> Notify User
```

Initial workflows:

## Workflow 1 — Interaction Completed

```text
Webhook
 -> Receive Interaction ID
 -> Get Interaction
 -> Generate Summary
 -> Extract Action Items
 -> Store Summary
 -> Create Task if required
 -> Notify User
```

## Workflow 2 — Important Interaction

```text
Interaction Completed
 -> Check Priority
 -> If High/Urgent
 -> Notify User
```

## Workflow 3 — Daily Task Summary

Future:

```text
Schedule
 -> Get Pending Tasks
 -> AI Summary
 -> Notify User
```

n8n should be self-hosted during development where practical.

---

# 25. Interaction Report

Report structure:

```text
Interaction Report
------------------

Participant:
Ahmed

Date:
2026-XX-XX

Duration:
04:32

Topic:
Production API issue

Priority:
High

Summary:
Participant reported HTTP 500 errors from the production API.

Key Details:
- Issue started around 10:15
- Affected API: ...
- Business impact: ...

Action Items:
1. Investigate API
2. Contact Ahmed

Status:
Pending
```

The report must be concise.

---

# 26. Representative Mode Rules

Representative Mode must have explicit boundaries.

AURA can:

- State that the user is unavailable
- Capture a message
- Answer using authorized knowledge
- Ask clarification questions
- Create a task if authorized
- Record interaction
- Generate a report

AURA should NOT automatically:

- Make irreversible decisions
- Delete important data
- Commit financial/legal actions
- Send sensitive information
- Claim to be a human
- Invent information

Potentially important actions should require confirmation.

---

# 27. AI Safety Principles

1. Never expose private knowledge to unauthorized users.
2. Never execute a tool without authorization.
3. Validate tool parameters.
4. Do not trust LLM-generated parameters blindly.
5. Require confirmation for risky actions.
6. Limit tool execution loops.
7. Log tool execution.
8. Do not expose internal prompts or hidden reasoning.
9. Clearly distinguish AI-generated summaries from original transcripts.
10. Fail safely when uncertain.

---

# 28. UI State Model

Assistant states:

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

The Angular UI should visibly communicate the current safe high-level state.

Example:

```text
🎙 Listening...
🔎 Searching knowledge...
⚙ Creating task...
🔊 Speaking...
```

---

# 29. Error Handling

Examples:

## STT failure

UI:
"Unable to understand the audio. Please try again."

## LLM failure

UI:
"AURA is temporarily unable to respond."

## RAG failure

AURA should say it could not access the knowledge base rather than inventing an answer.

## Tool failure

Example:
"Your task could not be created because the task service is unavailable."

## n8n failure

Interaction should still be stored. Automation failure should not destroy the interaction record.

---

# 30. Logging / Observability

Important events:

```text
Request received
Conversation created
LLM request
LLM response
RAG query
Retrieved document IDs
Tool selected
Tool executed
Tool failed
Interaction created
Interaction completed
n8n triggered
n8n failed
STT failure
TTS failure
```

Never log secrets or sensitive credentials.

---

# 31. Free/Local-First Technology Strategy

The project should avoid mandatory paid AI subscriptions.

Use local/open-source or free-tier services where practical.

Exact provider selection should be checked at implementation time because free tiers and availability change.

Candidate categories:

## LLM

Preferred strategy:
- Local LLM through Ollama where machine resources permit.
- Use an OpenAI-compatible API abstraction so a free-tier/cloud provider can be substituted.

Potential models:
- Llama-family instruct models
- Qwen-family instruct models
- Gemma-family models

## STT

Candidate:
- Whisper / faster-whisper locally

## TTS

Candidate:
- Piper
- Coqui-based/open TTS alternatives where compatible

## Embeddings

Candidate:
- sentence-transformers
- BGE-family embeddings
- other local embedding models

## Vector Database

Candidate:
- Qdrant
- Chroma
- PostgreSQL + pgvector if infrastructure simplicity becomes more important

Preferred starting candidate:
Qdrant if a dedicated vector DB is desired.

## Automation

n8n self-hosted.

## Containers

Docker / Docker Compose.

## Development

Angular 17
.NET 8
MySQL
Git/GitHub
Swagger/OpenAPI

Exact AI provider/model decisions must be validated before implementation rather than assumed.

---

# 32. Recommended Initial Infrastructure

Development environment:

```text
Windows
 |
+-- Angular 17
+-- .NET 8
+-- MySQL
+-- Docker
|    +-- Qdrant
|    +-- n8n
|
+-- Local LLM runtime if hardware supports it
|
+-- Git
```

Do not add Kubernetes, microservices, Kafka, Redis, or cloud infrastructure unless a real requirement emerges.

This project is intentionally a modular monolith for MVP.

---

# 33. Overall Technical Flow

```text
                         USER
                           |
                    Angular 17 UI
                           |
                 +---------+---------+
                 |                   |
              HTTP               WebSocket
                 |                   |
                 +---------+---------+
                           |
                       .NET 8 API
                           |
                   Agent Orchestrator
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
           LLM           RAG           Tools
             |             |             |
             |             v             v
             |        Embeddings      Task Service
             |             |             |
             |        Vector DB        MySQL
             |                           |
             +-------------+-------------+
                           |
                     Final Response
                           |
                    +------+------+
                    |             |
                   TTS        Conversation
                    |             |
                 Speaker        MySQL
                                  |
                                  v
                              n8n Webhook
                                  |
                   +--------------+--------------+
                   |              |              |
                Summary       Action Items    Notification
                   |              |
                   +------+-------+
                          |
                         User
```

---

# 34. Complete Representative Technical Flow

```text
External Participant
        |
        v
Representative UI
        |
        v
Voice/STT
        |
        v
.NET Agent
        |
        +---- Knowledge needed?
        |          |
        |         Yes
        |          |
        |         RAG
        |          |
        |       Vector DB
        |
        +---- Action needed?
        |          |
        |         Yes
        |          |
        |       Tool Registry
        |          |
        |       .NET Service
        |          |
        |         MySQL
        |
        v
AI Response
        |
       TTS
        |
        v
Participant
        |
        v
Interaction Complete
        |
        v
n8n
        |
        +--> Summary
        +--> Action Items
        +--> Task
        +--> Notification
        |
        v
User
        |
        v
"What happened?"
        |
        v
AURA
        |
        v
Interaction Report
```

---

# 35. Implementation Phases

## Phase 0 — Specification

Deliverables:
- BRS
- Architecture
- Database design
- UI wireframes
- Technology decisions
- Git repository strategy

## Phase 1 — Foundation

Build:
- Angular project
- .NET API
- MySQL
- Basic layout
- Health check
- Swagger
- Git

Acceptance:
Angular -> .NET -> MySQL works.

## Phase 2 — Text Assistant

Build:
- Chat UI
- Conversation API
- LLM integration
- Conversation persistence

Acceptance:
User can have a text conversation with AURA.

## Phase 3 — Agent

Build:
- Agent orchestrator
- Tool registry
- CreateTask
- GetTasks
- UpdateTask
- CompleteTask

Acceptance:
Natural language can execute task operations.

## Phase 4 — RAG

Build:
- Knowledge UI
- Document upload
- Text extraction
- Chunking
- Embeddings
- Vector DB
- Retrieval
- Source references

Acceptance:
AURA answers questions using uploaded documents.

## Phase 5 — Voice

Build:
- Microphone
- STT
- TTS
- WebSocket events
- Voice state UI

Acceptance:
User can talk to AURA and hear response.

## Phase 6 — Interaction / Representative

Build:
- Representative screen
- Availability
- Interaction recording
- Transcript
- Summary
- Action items

Acceptance:
Simulated external participant can interact with AURA.

## Phase 7 — n8n

Build:
- n8n
- Webhook
- Interaction processing workflow
- Summary/action workflow
- Notification workflow

Acceptance:
Completed interaction automatically triggers workflow.

## Phase 8 — Polish

Build:
- Authentication
- Authorization
- Error handling
- Logging
- Loading states
- UI refinement
- Docker
- README
- Architecture documentation

## Phase 9 — Demo

Prepare one end-to-end scenario:

```text
Umar unavailable
      ->
Person interacts with AURA
      ->
AURA understands issue
      ->
AURA asks questions
      ->
Interaction recorded
      ->
n8n processes interaction
      ->
Summary + action item
      ->
Umar asks what happened
      ->
AURA explains
```

---

# 36. Recommended Implementation Order

Do NOT start with voice.

Correct order:

```text
1. Repository + architecture
          |
2. Angular shell
          |
3. .NET API
          |
4. MySQL
          |
5. Text chat
          |
6. LLM
          |
7. Conversation persistence
          |
8. Tool calling
          |
9. RAG
          |
10. Voice
          |
11. Interaction/Representative
          |
12. n8n
          |
13. Security/error handling
          |
14. Demo/polish
```

Reason:
Voice introduces several moving parts. Building text AI first gives a stable agent core that voice can later use.

---

# 37. First Implementation Task

The first coding task is NOT an AI call.

Create the development foundation.

### Step 1

Create Git repository:

```text
AURA
```

### Step 2

Create documentation:

```text
/docs
    BRS.md
    ARCHITECTURE.md
    DATABASE.md
    API.md
    AI.md
```

### Step 3

Create Angular 17 application.

### Step 4

Create .NET 8 Web API.

### Step 5

Create MySQL database.

### Step 6

Create first API:

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

### Step 7

Connect Angular to the health API.

Expected:

```text
Angular
   |
   v
.NET /api/health
   |
   v
Healthy
```

This proves the basic application foundation.

---

# 38. Definition of Done

A feature is not complete merely because code compiles.

A feature is done when:

- Requirement exists
- UI exists where applicable
- API exists where applicable
- Database changes exist where applicable
- Validation exists
- Error handling exists
- Logging exists where useful
- Tested manually/automatically
- Code is committed
- Documentation is updated

---

# 39. Interview Talking Points

The final project should allow the developer to confidently discuss:

### Angular

- Feature-based architecture
- RxJS
- WebSocket
- State handling
- HTTP interceptors
- Route guards
- Component communication

### .NET

- Clean architecture
- Dependency injection
- REST APIs
- Async programming
- Middleware
- DTOs
- Validation
- Repository/service patterns

### AI

- LLM
- Prompt design
- Context windows
- Tool calling
- Agent orchestration
- Hallucination handling

### RAG

- Document ingestion
- Chunking
- Embeddings
- Vector search
- Retrieval
- Grounded generation

### Voice

- STT
- TTS
- Audio pipeline
- Real-time communication

### Automation

- Webhooks
- n8n
- Event-driven workflow
- AI-powered automation

### Architecture

- Modular monolith
- Separation of concerns
- External-provider abstraction
- Security
- Observability

---

# 40. Scope Guardrails

Do not add technology just to make the project look impressive.

Avoid initially:

- Kubernetes
- Microservices
- Kafka
- Complex event buses
- Multiple databases without need
- 20+ AI agents
- 50 tools
- Complex cloud infrastructure
- Real telephony before core representative workflow works

The objective is:

> Strong implementation depth > number of technologies.

---

# 41. Final MVP Definition

AURA MVP is complete when this scenario works end-to-end:

```text
User/Participant
       |
       v
Voice interaction
       |
       v
STT
       |
       v
AI Agent
       |
       +------ RAG when required
       |
       +------ Tool Calling when required
       |
       v
AI Response
       |
       v
TTS
       |
       v
Participant
       |
       v
Interaction saved
       |
       v
n8n triggered
       |
       +---- Summary
       +---- Action Items
       +---- Task
       +---- Notification
       |
       v
User asks:
"What happened?"
       |
       v
AURA gives report
```

This is the primary demonstration of the product.

---

# 42. Development Principle

Build AURA incrementally.

Every phase must leave the system in a runnable state.

```text
Foundation
   ->
Text AI
   ->
Agent
   ->
RAG
   ->
Voice
   ->
Representative
   ->
Automation
   ->
Production-quality polish
```

The architecture must remain open for future features, but the MVP must have a clear stopping point.

---

# 43. Master Project Statement

Use this as the canonical description when giving AURA context to another AI development tool:

> AURA is a voice-first AI Personal Representative built using Angular 17 and .NET 8. It combines LLM-based conversational AI, RAG over private user knowledge, AI tool/function calling, voice STT/TTS, MySQL persistence, vector search, WebSocket-based real-time interaction, and n8n workflow automation. The core business purpose is to allow an AI agent to assist the user and eventually represent the user during unavailable periods, capture external interactions, execute authorized actions, and provide concise interaction reports afterward. The MVP focuses on Voice + LLM + RAG + Tool Calling + Interaction Recording + AI Summarization + n8n Automation. Real telephone, WhatsApp, email, calendar, interviewer, and meeting integrations are future extensions and must not delay the MVP. The architecture should use a modular monolith with clean separation between Angular UI, .NET application/domain/infrastructure layers, AI services, RAG/vector storage, MySQL, and n8n. The project is primarily a portfolio/learning project for demonstrating practical enterprise software engineering and modern AI integration, with a free/local-first technology strategy wherever practical.
