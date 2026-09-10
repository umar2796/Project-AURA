# AURA — AI Technology Stack Decision

**Document:** `docs/AI.md`  
**Version:** 1.0  
**Status:** Development Baseline  
**Project:** AURA — AI Personal Representative

---

## 1. Purpose

This document defines the AI and supporting technology decisions for AURA.

The objective is to establish a stable technical foundation before application development begins.

AURA follows a **free/local-first** strategy wherever reasonably practical. AI providers and infrastructure components should remain replaceable through application-level abstractions.

The AURA BRS explicitly requires LLM, STT, TTS, embeddings, vector storage, RAG, tool calling, voice interaction and n8n automation. Exact provider selection is intentionally treated as an implementation-time technology decision. 

---

# 2. Technology Decisions

| Category | Selected Technology |
|---|---|
| LLM Runtime | Ollama |
| Primary LLM | Qwen3 8B |
| STT | faster-whisper |
| TTS | Piper |
| Embedding Model | BAAI BGE-M3 |
| Vector Database | Qdrant |
| Automation | n8n |
| Relational Database | MySQL 8.4 |
| Backend | .NET 8 Web API |
| Frontend | Angular 17 |
| Real-time Communication | ASP.NET Core SignalR |
| Authentication | ASP.NET Core Identity + JWT |
| Containerization | Docker Compose |
| Architecture | Modular Monolith |

---

# 3. Design Principles

## 3.1 Local-first

Where practical, AI processing should happen locally.

Preferred:

```text
Application
    |
    v
Local AI Service
    |
    v
Local Model
```

rather than making every AI operation dependent on a paid external API.

Benefits:

- No mandatory AI subscription
- Better control over private data
- Useful for learning AI infrastructure
- Reduced dependency on external providers
- Easier experimentation

---

## 3.2 Provider independence

AURA must not be tightly coupled to a specific AI provider.

The application should depend on interfaces such as:

```text
IAiChatService
IAiEmbeddingService
ISttService
ITtsService
IVectorStore
```

Implementations belong to the infrastructure layer.

Example:

```text
IAiChatService
      |
      +---- OllamaAiChatService
      |
      +---- FutureCloudAiService
      |
      +---- FutureOtherLocalAiService
```

This allows the underlying model/provider to be replaced without rewriting the agent.

---

# 4. LLM Decision

## Selected

**Ollama + Qwen3 8B**

### Responsibilities

The LLM is responsible for:

- Understanding user messages
- Generating conversational responses
- Determining when knowledge retrieval is required
- Selecting tools
- Extracting tool parameters
- Processing tool results
- Generating final responses
- Summarizing interactions
- Extracting action items

This matches the BRS's defined LLM responsibilities.

---

## Architecture

```text
Angular
   |
   v
.NET Agent
   |
   v
IAiChatService
   |
   v
Ollama
   |
   v
Qwen3 8B
```

The LLM must not directly access MySQL.

Instead:

```text
LLM
 |
Tool Request
 |
.NET Agent
 |
Authorized Tool
 |
Application Service
 |
Repository
 |
MySQL
```

---

## Why Qwen3 8B?

Qwen3 8B is selected as the initial general-purpose model because AURA needs more than simple text generation.

The model must support the broader agent workflow:

```text
Conversation
     +
RAG
     +
Tool Calling
     +
Structured Parameters
     +
Summarization
```

AURA's tool-calling architecture is particularly important because task operations are core MVP functionality.

---

## Alternative

### Smaller Qwen model

A smaller model can be used if local hardware cannot provide acceptable performance.

The application should therefore treat the model name as configuration rather than application logic.

Example:

```text
AI_MODEL=qwen3:8b
```

could later become:

```text
AI_MODEL=<smaller-model>
```

without changing the agent architecture.

---

## Trade-offs

### Advantages

- Local execution
- No mandatory paid API
- Good learning value
- Provider can be replaced
- Suitable for experimentation

### Disadvantages

- Requires sufficient local hardware
- Larger models require more RAM/VRAM
- Local inference may be slower than cloud APIs
- Model quality can differ from larger hosted models

---

# 5. STT Decision

## Selected

**faster-whisper**

Speech-to-text converts microphone audio into text that can be processed by the AURA agent.

---

## Flow

```text
Browser Microphone
       |
       v
Audio
       |
       v
STT Service
       |
       v
faster-whisper
       |
       v
Text
       |
       v
AURA Agent
```

---

## Why local STT?

AURA is intended to support local/free AI processing where practical.

Local STT provides:

- No mandatory API cost
- Better control of audio data
- Offline/local development capability
- Practical hands-on experience with speech processing

---

## Alternative

Cloud speech-to-text APIs can be introduced later if they provide significantly better latency or accuracy.

The application should therefore use:

```text
ISttService
```

rather than calling faster-whisper directly from controllers.

---

# 6. TTS Decision

## Selected

**Piper**

Piper will convert AURA's text response into speech.

---

## Flow

```text
AURA Agent
    |
    v
Text Response
    |
    v
ITtsService
    |
    v
Piper
    |
    v
Audio
    |
    v
Browser
```

---

## Why Piper?

Primary reasons:

- Local execution
- No mandatory paid TTS service
- Suitable for a voice-first application
- Keeps the initial voice pipeline self-contained

---

## Alternative

A cloud TTS provider may be considered later if voice quality or streaming performance becomes a significant product requirement.

The application should remain provider-independent through:

```text
ITtsService
```

---

# 7. Embedding Decision

## Selected

**BAAI BGE-M3**

The embedding model converts text into numerical vectors.

Example:

```text
"UCC validation is required..."
             |
             v
        Embedding Model
             |
             v
       [0.023, -0.81, ...]
```

These vectors are stored in Qdrant.

---

## Document ingestion

```text
Document
   |
   v
Text Extraction
   |
   v
Chunking
   |
   v
BGE-M3
   |
   v
Vector
   |
   v
Qdrant
```

---

## Query

```text
User Question
      |
      v
BGE-M3
      |
      v
Query Vector
      |
      v
Qdrant
      |
      v
Relevant Chunks
```

---

## Why local embeddings?

Embedding generation can be performed locally and does not need to depend on a paid embedding API.

This is especially useful because AURA's RAG pipeline may process private documents.

---

# 8. Vector Database Decision

## Selected

**Qdrant**

Qdrant will be responsible for semantic/vector search.

MySQL remains the system of record for relational application data.

---

## Separation

```text
MySQL
 |
 +-- Users
 +-- Conversations
 +-- Messages
 +-- Tasks
 +-- Interactions
 +-- KnowledgeDocuments
 +-- ActionItems


Qdrant
 |
 +-- Knowledge vectors
 +-- Vector metadata
```

---

## Why Qdrant?

The BRS explicitly requires a vector database for RAG.

Qdrant gives AURA a dedicated vector-search layer rather than treating vector data as ordinary relational data.

This also provides useful practical experience with:

- Vector embeddings
- Similarity search
- Metadata filtering
- Top-K retrieval
- Retrieval quality
- RAG architecture

---

## Required metadata

Each stored chunk should contain metadata similar to:

```json
{
  "userId": "...",
  "documentId": "...",
  "fileName": "UCC_BRS.pdf",
  "chunkIndex": 12
}
```

The exact schema can be finalized during RAG implementation.

---

## Critical security rule

Every knowledge query must enforce ownership.

Conceptually:

```text
Question
   |
   v
Embedding
   |
   v
Qdrant
   |
   +---- UserId filter
   |
   v
Allowed chunks only
```

AURA must never retrieve another user's private knowledge.

---

# 9. RAG Decision

RAG is not a separate product/service. It is an application capability built using:

```text
Document Processing
+
BGE-M3
+
Qdrant
+
LLM
```

---

## Ingestion pipeline

```text
PDF / TXT / Markdown
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
BGE-M3
        |
        v
Qdrant
```

---

## Query pipeline

```text
User Question
      |
      v
Question Embedding
      |
      v
Qdrant Search
      |
      v
Top-K Chunks
      |
      v
Prompt + Context
      |
      v
Qwen3
      |
      v
Grounded Answer
```

---

## RAG responsibilities

The implementation must address:

- Chunk size
- Chunk overlap
- Embedding dimensions
- Similarity metric
- Top-K retrieval
- Metadata filtering
- Source references
- Context limits
- Retrieval failure
- Hallucination control

These concepts are explicitly part of the AURA learning objectives.

---

# 10. Agent / Tool Calling Decision

AURA will use an **Agent Orchestrator** in the .NET application.

The LLM does not execute application operations directly.

---

## Architecture

```text
User
 |
 v
Agent Orchestrator
 |
 v
LLM
 |
 +---- Final Answer
 |
 +---- Tool Call
          |
          v
     Tool Registry
          |
          v
     Authorization
          |
          v
     Application Service
          |
          v
        MySQL
```

---

## Initial tools

```text
CreateTask
GetTasks
UpdateTask
CompleteTask
GetInteraction
```

Additional tools may be added later.

---

## Tool execution rules

Every tool must:

1. Validate parameters
2. Identify the current user
3. Check authorization
4. Execute the application operation
5. Return a structured result
6. Log the execution

The LLM-generated parameters must never be trusted blindly.

---

# 11. n8n Decision

## Selected

**Self-hosted n8n**

n8n will be the workflow automation layer.

It should not replace the .NET backend.

---

## Responsibility split

### .NET

Responsible for:

- Business logic
- Authorization
- Agent orchestration
- Database operations
- Core application services

### n8n

Responsible for:

- Workflow automation
- Webhooks
- Notifications
- Multi-step automation
- Post-interaction processing

---

## Example

```text
Interaction Completed
        |
        v
.NET
        |
        | Webhook
        v
      n8n
        |
        +--> Generate Summary
        |
        +--> Extract Action Items
        |
        +--> Create Task
        |
        +--> Notify User
```

This follows the intended AURA architecture.

---

# 12. Voice Architecture Decision

AURA's voice pipeline will initially be:

```text
                 USER
                   |
                   v
             Microphone
                   |
                   v
              Audio Input
                   |
                   v
          faster-whisper
                   |
                   v
              Text Input
                   |
                   v
          Agent Orchestrator
                   |
          +--------+--------+
          |                 |
         RAG              Tools
          |                 |
          +--------+--------+
                   |
                   v
                Qwen3
                   |
                   v
             Text Response
                   |
                   v
                 Piper
                   |
                   v
              Audio Output
                   |
                   v
                Speaker
```

---

## MVP limitation

We will **not initially build full duplex streaming voice**.

The first version should prove:

```text
Speak
  ↓
Transcribe
  ↓
Process
  ↓
Respond
  ↓
Speak
```

Streaming optimization can be added later.

This follows the BRS principle of keeping optional complexity from delaying the MVP.

---

# 13. Real-Time Communication Decision

## Selected

**ASP.NET Core SignalR**

SignalR will be used for real-time conversation/application events.

HTTP remains the default mechanism for CRUD operations.

---

## HTTP

Used for:

```text
Tasks
Knowledge
Conversations
Interactions
Settings
Authentication
```

---

## SignalR

Used for:

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

---

## Architecture

```text
Angular
   |
   | SignalR
   v
.NET
   |
   v
Agent / Voice Services
   |
   v
Real-time Events
   |
   v
Angular
```

This allows the UI to communicate safe high-level states without exposing internal reasoning.

---

# 14. Authentication Decision

## Selected

**ASP.NET Core Identity + JWT**

Authentication is required because AURA contains private:

- Conversations
- Tasks
- Knowledge documents
- Interactions
- Memories
- Agent configuration

---

## Flow

```text
Angular
   |
   v
Login
   |
   v
ASP.NET Core Identity
   |
   v
JWT
   |
   v
Angular
   |
   v
Authenticated API Requests
```

The JWT identifies the authenticated user.

Backend services must use the authenticated identity rather than trusting a UserId supplied by the client.

---

# 15. MySQL Decision

## Selected

**MySQL**

MySQL will remain the primary relational database.

It will store:

```text
Users
Conversations
Messages
Tasks
Interactions
InteractionMessages
ActionItems
KnowledgeDocuments
AgentMemories
```

---

## Important distinction

```text
MySQL
    =
Application/System of Record


Qdrant
    =
Semantic Retrieval
```

The two databases have different responsibilities.

---

# 16. Docker Decision

## Selected

**Docker Compose**

Docker will initially be used for infrastructure services rather than forcing the entire development environment into containers.

Initial target:

```text
Docker Compose
    |
    +-- MySQL
    +-- Qdrant
    +-- n8n
```

Angular and .NET can initially run directly on the development machine.

Ollama can also run locally outside the Compose environment.

---

## Why?

This keeps the development setup understandable.

We want to learn:

```text
AI
+
RAG
+
Agents
+
Voice
+
n8n
+
Angular
+
.NET
```

We do not want Docker itself to become the primary development problem.

---

# 17. Overall AI Architecture

```text
                         AURA
                           |
                      Angular 17
                           |
                  HTTP + SignalR
                           |
                        .NET 8
                           |
                  Agent Orchestrator
                           |
              +------------+------------+
              |            |            |
              v            v            v
             LLM          RAG         Tools
              |            |            |
              v            v            v
           Ollama       Qdrant      Application
              |            |         Services
           Qwen3 8B       ^              |
                           |              v
                        BGE-M3          MySQL
                           |
                     Knowledge Data


Voice:

Audio
  |
  v
faster-whisper
  |
  v
Text
  |
  v
Agent
  |
  v
Text
  |
  v
Piper
  |
  v
Audio


Automation:

.NET
 |
 v
n8n
 |
 +--> Summary
 +--> Action Items
 +--> Task
 +--> Notification
```

---

# 18. Provider Abstraction

The infrastructure must expose interfaces instead of allowing application code to depend directly on third-party implementations.

Recommended interfaces:

```csharp
IAiChatService
IAiEmbeddingService
ISttService
ITtsService
IVectorStore
IN8nService
```

Possible infrastructure implementations:

```text
OllamaAiChatService
BgeEmbeddingService
FasterWhisperSttService
PiperTtsService
QdrantVectorStore
N8nService
```

This is an important architectural decision.

---

# 19. Configuration Strategy

Secrets and provider configuration must not be hardcoded.

Examples:

```text
AI_PROVIDER=ollama
AI_MODEL=qwen3:8b

QDRANT_URL=...

MYSQL_CONNECTION_STRING=...

N8N_BASE_URL=...

JWT_SECRET=...
```

Actual configuration names may be adjusted during implementation.

Development secrets should be stored using appropriate local configuration/secrets mechanisms and must never be committed to Git.

---

# 20. Error Strategy

AI services are external dependencies from the application's perspective even when they run locally.

Therefore:

```text
LLM failure
   ↓
Meaningful application error


STT failure
   ↓
Ask user to retry


TTS failure
   ↓
Display text response


Qdrant failure
   ↓
Do not fabricate a RAG answer


n8n failure
   ↓
Keep interaction record
   +
Log automation failure
```

A failure in one AI component must not unnecessarily destroy already-persisted business data.

---

# 21. Security Principles

AURA must:

- Authenticate protected APIs
- Authorize tool execution
- Validate tool parameters
- Filter RAG data by user
- Protect private documents
- Avoid exposing prompts/internal reasoning
- Avoid logging secrets
- Audit important tool executions
- Require confirmation for risky actions
- Fail safely when uncertain

The LLM is treated as an intelligent decision component, **not as a trusted security boundary**.

---

# 22. Why We Are Not Using More Technologies

The following are deliberately excluded from the MVP:

```text
Kubernetes
Kafka
Microservices
Redis
Complex event buses
Multiple LLM agents
Cloud infrastructure
Real telephony
WhatsApp integration
Email integration
Calendar integration
```

This is intentional.

AURA's BRS explicitly states that technologies should not be added merely to make the project appear impressive and that the project should prioritize implementation depth over the number of technologies. 

The target architecture is therefore:

> **Modular monolith + replaceable AI infrastructure**

---

# 23. MVP AI Scope

The first complete AI milestone should eventually demonstrate:

```text
User
 |
 v
Text / Voice
 |
 v
AURA Agent
 |
 +---- Normal Answer
 |
 +---- RAG
 |
 +---- Tool Calling
 |
 v
Response
 |
 v
Conversation History
```

Then:

```text
Interaction
 |
 v
n8n
 |
 +--> Summary
 +--> Action Items
 +--> Task
 +--> Notification
```

This is enough to demonstrate the core AI engineering capabilities expected from AURA.

---

# 24. Future Replacement Strategy

The initial stack is not intended to permanently lock AURA to specific providers.

Potential future replacements:

```text
Qwen3
   ↓
Another local model
   ↓
Cloud LLM


faster-whisper
   ↓
Another local STT
   ↓
Cloud STT


Piper
   ↓
Another local TTS
   ↓
Cloud TTS


Qdrant
   ↓
Another vector store
```

The application should continue working because the core application depends on interfaces rather than concrete providers.

---

# 25. Final Decision

The AURA v1.0 technology baseline is:

```text
Frontend
    Angular 17

Backend
    .NET 8 Web API

Relational Database
    MySQL

LLM Runtime
    Ollama

LLM
    Qwen3 8B

STT
    faster-whisper

TTS
    Piper

Embeddings
    BAAI BGE-M3

Vector Database
    Qdrant

Automation
    Self-hosted n8n

Real-time
    ASP.NET Core SignalR

Authentication
    ASP.NET Core Identity + JWT

Containerization
    Docker Compose

Architecture
    Modular Monolith

AI Strategy
    Local-first + provider abstraction
```

## Status

**Decision: Approved as the AURA development baseline.**

The stack can be revisited if practical testing shows that local hardware, model quality, latency, or compatibility makes one component unsuitable.

No AI provider should be changed merely for novelty. A replacement should have a concrete technical reason.

---

# 26. Next Engineering Milestone

With the AI stack decision complete, **we should now stop making technology decisions and start building the foundation.**

The next sequence is:

```text
AURA/
│
├── aura-client/
├── aura-api/
├── aura-workflows/
├── docs/
│   ├── BRS.md
│   ├── ARCHITECTURE.md
│   ├── DATABASE.md
│   ├── AI.md
│   └── API.md
└── README.md
```

Then:

```text
Git repository
      ↓
Angular 17
      ↓
.NET 8
      ↓
MySQL
      ↓
GET /api/health
      ↓
Angular displays Healthy
```

Only after this foundation is working will we connect Ollama and build the first text-AI capability.