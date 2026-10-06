# Implementation Tasks — AI Business Operations Agent

**Version:** 0.1
**Status:** Draft

## 1. Development Strategy

The project will be implemented incrementally.

Each task must:

* Have a clear objective.
* Modify a limited part of the system.
* Be independently testable whenever possible.
* Respect the architecture defined in `architecture.md`.
* Avoid implementing future functionality that is not required by the MVP.

Implementation order:

```text
Foundation
    ↓
Backend
    ↓
Database
    ↓
Authentication
    ↓
Customers
    ↓
Knowledge Base
    ↓
AI Service
    ↓
Agent Tools
    ↓
Human Approval
    ↓
Email
    ↓
Audit
    ↓
Frontend
    ↓
Integration
```

---

# 2. Phase 1 — Project Foundation

## TASK-001 — Initialize repository structure

### Goal

Create the initial monorepo structure.

### Structure

```text
ai-business-agent/
├── apps/
│   ├── frontend/
│   ├── backend/
│   └── ai-service/
├── specs/
├── docker/
├── .env.example
├── .gitignore
└── README.md
```

### Requirements

* Use Git.
* Add appropriate `.gitignore`.
* Add basic README.
* Do not implement application functionality.

### Acceptance criteria

* Repository has the expected structure.
* Git status is clean after commit.
* README explains the project at a high level.

---

# 3. Phase 2 — Backend Foundation

## TASK-002 — Initialize NestJS backend

### Goal

Create the NestJS application.

### Requirements

* TypeScript.
* NestJS.
* Environment configuration.
* Global validation.
* Basic health endpoint.

### Acceptance criteria

```http
GET /health
```

returns:

```json
{
  "status": "ok"
}
```

---

## TASK-003 — Configure PostgreSQL

### Goal

Connect NestJS to PostgreSQL.

### Requirements

* PostgreSQL.
* Docker Compose for local development.
* Environment-based database configuration.
* ORM configuration.

### Acceptance criteria

* PostgreSQL starts locally.
* NestJS successfully connects.
* Application fails clearly if database configuration is invalid.

---

# 4. Phase 3 — Database Foundation

## TASK-004 — Create organization model

Create:

```text
Organization
OrganizationMember
User
```

### Requirements

Support:

* organization creation
* membership
* roles

Initial roles:

```text
ADMIN
MEMBER
```

### Acceptance criteria

* Database migrations work.
* Relations are defined.
* Organization membership can be persisted.

---

# 5. Phase 4 — Authentication

## TASK-005 — Implement authentication

### Goal

Allow users to register and authenticate.

### Requirements

* Password hashing.
* Login.
* Authentication token/session.
* Authenticated request context.

### Acceptance criteria

A user can:

1. Register.
2. Log in.
3. Access an authenticated endpoint.
4. Be rejected when unauthenticated.

---

## TASK-006 — Implement organization authorization

### Goal

Ensure users can only access organizations they belong to.

### Requirements

Every authenticated request must have organization context.

### Acceptance criteria

A user from Organization A cannot access Organization B resources.

---

# 6. Phase 5 — Customer Management

## TASK-007 — Create customer model

Customer fields:

```text
id
organization_id
name
email
phone
status
created_at
updated_at
```

### Acceptance criteria

Customers belong to an organization.

---

## TASK-008 — Customer CRUD

Implement:

```http
POST   /customers
GET    /customers
GET    /customers/:id
PATCH  /customers/:id
DELETE /customers/:id
```

### Requirements

* Tenant isolation.
* Validation.
* Pagination for customer listing.
* Search by name/email.

### Acceptance criteria

A user can manage customers within their organization.

---

## TASK-009 — Customer purchases and balances

Create customer purchase/debt information.

The system must allow the agent to identify customers with outstanding balances.

### Acceptance criteria

The backend can retrieve:

* customer purchases
* total outstanding balance
* overdue information

---

# 7. Phase 6 — Knowledge Base

## TASK-010 — Document upload

Implement document upload.

Initial supported format:

```text
PDF
```

### Requirements

* Store document metadata.
* Store uploaded file.
* Associate document with organization.
* Validate file type and size.

---

## TASK-011 — Document text extraction

Create the document processing pipeline:

```text
PDF
 ↓
Text extraction
 ↓
Normalized text
```

### Acceptance criteria

The system can extract readable text from a valid PDF.

---

## TASK-012 — Document chunking

Split extracted text into chunks suitable for retrieval.

Each chunk must retain:

* document ID
* organization ID
* content
* metadata

---

## TASK-013 — Generate embeddings

Generate embeddings for document chunks.

### Requirements

* Embeddings generated through the AI service.
* Store embeddings in PostgreSQL using pgvector.

---

## TASK-014 — Semantic search

Implement knowledge-base search.

Input:

```text
query
organization_id
```

Output:

```text
relevant document chunks
```

### Critical requirement

Search must never return chunks belonging to another organization.

---

# 8. Phase 7 — Python AI Service

## TASK-015 — Initialize FastAPI AI service

Create:

```text
apps/ai-service/
```

### Requirements

* Python.
* FastAPI.
* Environment configuration.
* Health endpoint.

Acceptance:

```http
GET /health
```

returns:

```json
{
  "status": "ok"
}
```

---

## TASK-016 — LLM integration

Create a provider abstraction for the LLM.

The AI service must be able to send:

* system instructions
* user messages
* contextual information

and receive a structured response.

Do not expose API keys to NestJS or the frontend.

---

# 9. Phase 8 — Agent

## TASK-017 — Agent basic conversation

Create the first agent endpoint:

```http
POST /ai/agent/run
```

Input:

```json
{
  "organizationId": "...",
  "userId": "...",
  "message": "..."
}
```

Output:

```json
{
  "message": "...",
  "action": null,
  "requiresApproval": false
}
```

---

## TASK-018 — Tool calling

Implement the initial tool interface.

Tools:

```text
search_customers
get_customer
search_knowledge_base
create_email_draft
send_email
```

The agent must select tools based on the user's request.

---

## TASK-019 — Customer tools

Implement:

```text
search_customers()
get_customer()
```

The tools must respect organization isolation.

Example:

```text
"What do we know about Juan Pérez?"
```

should retrieve the customer's information when available.

---

## TASK-020 — Knowledge-base tool

Implement:

```text
search_knowledge_base()
```

The agent must use it when company policies or internal documentation are required.

---

# 10. Phase 9 — Policy-Aware Agent

## TASK-021 — Apply company policies

The agent must combine:

```text
Customer data
+
Company knowledge
+
User request
```

to determine the appropriate response.

Example:

```text
Customer:
20 days overdue

Company policy:
>15 days → email reminder
```

Agent:

```text
"Juan Pérez has been overdue for 20 days.
According to the company policy, the next step is
to send an email reminder."
```

---

# 11. Phase 10 — Human Approval

## TASK-022 — Create approval model

Create:

```text
Approval
```

Fields should include:

```text
id
organization_id
requested_by
action_type
payload
status
created_at
approved_at
rejected_at
```

Statuses:

```text
PENDING
APPROVED
REJECTED
EXECUTED
FAILED
```

---

## TASK-023 — Action proposals

The AI agent must return structured action proposals.

Example:

```json
{
  "action": {
    "type": "SEND_EMAIL",
    "payload": {
      "customerId": "...",
      "subject": "...",
      "body": "..."
    }
  },
  "requiresApproval": true
}
```

---

## TASK-024 — Approval endpoints

Implement:

```http
GET  /approvals
GET  /approvals/:id
POST /approvals/:id/approve
POST /approvals/:id/reject
```

### Critical requirement

The backend must verify:

* authentication
* organization membership
* permissions
* approval status

before executing an action.

---

# 12. Phase 11 — Email

## TASK-025 — Email provider integration

Implement the email provider integration.

The backend must expose an internal email service.

The AI service must not directly send emails.

---

## TASK-026 — Send approved email

Flow:

```text
AI
 ↓
Action proposal
 ↓
Approval
 ↓
NestJS
 ↓
Email service
 ↓
External provider
```

Only approved actions can trigger email sending.

---

# 13. Phase 12 — Audit Trail

## TASK-027 — Audit log model

Create:

```text
AuditLog
```

Track:

* user
* organization
* action
* entity
* status
* metadata
* timestamp

---

## TASK-028 — Record agent activity

Record:

* agent request
* tool call
* action proposal
* approval
* rejection
* execution
* failure

---

# 14. Phase 13 — Conversations

## TASK-029 — Conversation model

Create:

```text
Conversation
Message
```

Store:

* user
* organization
* messages
* timestamps

---

## TASK-030 — Persist agent conversations

The frontend must be able to retrieve previous conversations.

The agent should receive relevant conversation context when processing a new message.

---

# 15. Phase 14 — Frontend

## TASK-031 — Initialize Next.js frontend

Create the frontend application.

Requirements:

* Next.js
* TypeScript
* Basic application layout
* Environment configuration

---

## TASK-032 — Authentication UI

Create:

* Login
* Register
* Logout

---

## TASK-033 — Customer UI

Create:

* Customer list
* Search
* Customer details
* Customer balance

---

## TASK-034 — Knowledge-base UI

Create:

* Document list
* PDF upload
* Processing status
* Document details

---

## TASK-035 — AI chat UI

Create the main agent interface.

The user must be able to:

* send messages
* receive responses
* see proposed actions
* see when approval is required

---

## TASK-036 — Approval UI

Display pending actions.

Example:

```text
Agent wants to send:

To: Juan Pérez
Subject: Recordatorio de pago

[Approve]
[Reject]
```

---

## TASK-037 — Audit UI

Create an execution history showing:

* action
* user
* status
* timestamp
* result

---

# 16. Phase 15 — Integration

## TASK-038 — Connect frontend and backend

Connect:

```text
Next.js
   ↓
NestJS
```

for:

* authentication
* customers
* documents
* conversations
* approvals
* audit logs

---

## TASK-039 — Connect backend and AI service

Connect:

```text
NestJS
   ↓
Python/FastAPI
```

for:

* agent execution
* embeddings
* RAG
* tool calling

---

# 17. Phase 16 — End-to-End MVP

## TASK-040 — Complete primary business flow

The following flow must work end-to-end:

```text
1. User creates organization
        ↓
2. Adds customers
        ↓
3. Uploads company policy PDF
        ↓
4. Document is processed
        ↓
5. User asks:
   "Which customers are more than
    15 days overdue?"
        ↓
6. Agent searches customers
        ↓
7. Agent retrieves company policy
        ↓
8. Agent determines appropriate action
        ↓
9. Agent proposes email
        ↓
10. User approves
        ↓
11. NestJS sends email
        ↓
12. Action is recorded in audit log
```

---

# 18. Testing Requirements

Every major backend module must include tests.

Minimum coverage areas:

* Authentication
* Authorization
* Tenant isolation
* Customer operations
* Document processing
* Semantic search
* Agent tool selection
* Approval rules
* Email execution
* Audit logging

Critical security rules must have automated tests.

Especially:

```text
User from Organization A
must never access
Organization B data.
```

And:

```text
Unapproved action
must never execute.
```

---

# 19. Definition of Done

The MVP is considered complete when:

* Users can authenticate.
* Organizations are isolated.
* Customers can be managed.
* Documents can be uploaded.
* Documents can be searched semantically.
* The AI agent can answer questions using company data.
* The agent can use tools.
* Company policies influence proposed actions.
* Side-effect actions require approval.
* Approved emails can be sent.
* Agent activity is auditable.
* The main business flow works end-to-end.
* Critical backend behavior has automated tests.
