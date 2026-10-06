# Architecture Spec — AI Business Operations Agent

**Version:** 0.1
**Status:** Draft

## 1. Architecture Goals

The architecture must:

* Support the MVP defined in `product.md`.
* Separate business logic from AI/agent logic.
* Support multiple organizations with strict data isolation.
* Allow the AI agent to use tools.
* Require human approval for actions that create, modify, or send information.
* Keep an auditable history of agent decisions and executions.
* Be simple enough to develop and deploy as an MVP.
* Allow future integrations without redesigning the entire system.

---

## 2. High-Level Architecture

```text
                         ┌──────────────────┐
                         │     Next.js      │
                         │    Frontend      │
                         └────────┬─────────┘
                                  │
                                  │ HTTP
                                  ▼
                         ┌──────────────────┐
                         │     NestJS       │
                         │   Backend API    │
                         └───────┬─────┬────┘
                                 │     │
                    ┌────────────┘     └──────────────┐
                    │                                 │
                    ▼                                 ▼
          ┌──────────────────┐             ┌──────────────────┐
          │   PostgreSQL     │             │   AI Service     │
          │   + pgvector     │             │   Python/FastAPI  │
          └──────────────────┘             └────────┬─────────┘
                                                    │
                                                    ▼
                                             ┌──────────────┐
                                             │     LLM      │
                                             └──────────────┘
```

### Main components

| Component      | Responsibility                                |
| -------------- | --------------------------------------------- |
| Next.js        | User interface                                |
| NestJS         | Application/business backend                  |
| PostgreSQL     | Persistent application data                   |
| pgvector       | Vector search for knowledge base              |
| Python/FastAPI | AI agent and AI processing                    |
| LLM            | Natural language reasoning and tool selection |

---

# 3. Frontend

## Technology

* Next.js
* TypeScript
* React

## Responsibilities

The frontend is responsible for:

* Authentication screens
* Organization interface
* Customer management
* Document management
* AI chat
* Approval interface
* Execution history
* Displaying errors and statuses

The frontend must **not** enforce business authorization by itself.

For example, hiding the "Approve" button is not considered sufficient security.

The backend must always validate whether the authenticated user can perform the operation.

---

# 4. Backend — NestJS

NestJS is the main application backend.

It owns the business rules and is the source of truth for application operations.

## Responsibilities

* Authentication
* Authorization
* Organizations
* Users and memberships
* Customers
* Customer purchases
* Documents
* Conversations
* Approvals
* Audit logs
* Email execution
* AI service orchestration
* Multi-tenant data isolation

## Example modules

```text
src/
├── auth/
├── organizations/
├── customers/
├── documents/
├── conversations/
├── approvals/
├── audit/
├── email/
└── ai/
```

---

# 5. AI Service — Python/FastAPI

Python is responsible for AI-specific functionality.

The AI service should not become the main application backend.

Its responsibility is to process AI-related operations.

## Responsibilities

* Agent orchestration
* Prompt construction
* LLM communication
* Tool selection
* RAG retrieval
* Embedding generation
* Context construction
* Agent responses
* Structured action proposals

## Example structure

```text
ai-service/
├── app/
│   ├── agents/
│   ├── tools/
│   ├── rag/
│   ├── llm/
│   └── main.py
└── tests/
```

---

# 6. Communication Between NestJS and Python

NestJS communicates with the Python AI service through HTTP.

Example:

```text
Next.js
   │
   │ POST /api/agent/chat
   ▼
NestJS
   │
   │ POST /ai/agent/run
   ▼
Python/FastAPI
   │
   ▼
LLM
```

The Python service should return structured responses rather than UI-specific responses.

Example:

```json
{
  "message": "Encontré 4 clientes con deuda superior a 15 días.",
  "action": null,
  "requiresApproval": false
}
```

For an action:

```json
{
  "message": "La política indica que corresponde enviar un email.",
  "action": {
    "type": "SEND_EMAIL",
    "customerId": "customer_123",
    "subject": "Recordatorio de pago",
    "body": "..."
  },
  "requiresApproval": true
}
```

---

# 7. Agent Architecture

The agent is responsible for understanding the user's request and selecting the appropriate tools.

Example:

```text
User
 │
 ▼
AI Agent
 │
 ├── search_customers()
 │
 ├── get_customer()
 │
 ├── search_knowledge_base()
 │
 └── create_email_draft()
```

The agent should not directly bypass NestJS business rules.

For operations that modify data or produce external side effects, the agent produces an action proposal.

NestJS decides whether the action is allowed and whether approval is required.

---

# 8. Tools

Initial tools:

```text
search_customers()
get_customer()
search_knowledge_base()
create_email_draft()
send_email()
```

### Read-only tools

These can be executed automatically:

* `search_customers`
* `get_customer`
* `search_knowledge_base`

### Side-effect tools

These require approval:

* `create_email_draft`
* `send_email`

The approval requirement must be enforced by the backend and must not depend only on the AI model.

---

# 9. Human Approval Flow

For a side-effect operation:

```text
User request
     │
     ▼
AI Agent
     │
     ▼
Determine action
     │
     ▼
Create action proposal
     │
     ▼
NestJS
     │
     ▼
Approval required
     │
     ▼
User approves
     │
     ▼
NestJS validates permissions
     │
     ▼
Execute action
     │
     ▼
Audit log
```

The AI service must never be considered the final authority for authorization.

---

# 10. Knowledge Base / RAG

Company documents will be processed into searchable chunks.

Initial flow:

```text
PDF
 │
 ▼
NestJS
 │
 ▼
Document storage
 │
 ▼
Python
 │
 ├── Extract text
 ├── Chunk text
 ├── Generate embeddings
 │
 ▼
PostgreSQL + pgvector
```

When the agent needs company knowledge:

```text
User question
      │
      ▼
Generate query embedding
      │
      ▼
pgvector similarity search
      │
      ▼
Relevant chunks
      │
      ▼
LLM context
      │
      ▼
Agent response
```

Business policies will initially be represented as documents in the knowledge base.

A future version may introduce structured policy entities.

---

# 11. Database

PostgreSQL will be the primary database.

Initial entities:

```text
users
organizations
organization_members
customers
customer_purchases
documents
document_chunks
conversations
messages
approvals
audit_logs
```

All organization-owned records must contain an `organization_id` or be reachable through an organization-owned relation.

---

# 12. Multi-Tenancy

The application will use a shared database with logical tenant isolation.

Every request must establish the authenticated user's organization context.

Example:

```text
User
 │
 ▼
Organization
 │
 ├── Customers
 ├── Documents
 ├── Conversations
 ├── Approvals
 └── Audit Logs
```

A user belonging to Organization A must never be able to access Organization B data.

Tenant isolation must be enforced by NestJS/backend queries and authorization rules.

---

# 13. Authentication and Authorization

The MVP will use application-managed authentication.

The backend will identify:

* User
* Organization
* Organization membership
* Role

Initial roles:

```text
ADMIN
MEMBER
```

Permissions will be enforced on the backend.

Example:

```text
ADMIN
 ├── manage organization
 ├── manage users
 ├── manage documents
 └── approve actions

MEMBER
 ├── view customers
 ├── use agent
 ├── upload documents
 └── request actions
```

---

# 14. Email Integration

Email sending will be handled by NestJS.

The AI agent can propose an email, but the backend is responsible for executing the external operation.

```text
AI
 │
 │ proposal
 ▼
NestJS
 │
 │ approval
 ▼
Email provider
```

This keeps external side effects outside the AI service.

---

# 15. Audit Trail

Every important agent operation must be recorded.

Examples:

* User request
* Tool selected
* Tool execution
* Action proposed
* Approval
* Rejection
* External action
* Error

Example:

```text
audit_logs

id
organization_id
user_id
action
entity_type
entity_id
status
metadata
created_at
```

This allows the application to answer:

> What did the agent do?

> Why did it do it?

> Who approved it?

> What happened after approval?

---

# 16. Error Handling

The system must distinguish between:

### AI errors

Examples:

* LLM unavailable
* invalid tool arguments
* malformed model response

### Application errors

Examples:

* unauthorized access
* customer not found
* invalid organization

### External integration errors

Examples:

* email provider unavailable
* email rejected
* timeout

AI failures must not silently execute external actions.

---

# 17. Security Principles

The MVP must follow these principles:

* Never trust the AI model for authorization.
* Never expose another organization's data.
* Validate tool parameters in the backend.
* Validate permissions before side effects.
* Require approval for external actions.
* Do not expose API keys to the frontend.
* Log important agent actions.
* Sanitize and validate uploaded documents.
* Keep secrets in environment variables.

---

# 18. MVP Deployment

The system should be deployable as separate services:

```text
Frontend
Next.js
   │
   ▼
Backend
NestJS
   │
   ├──────────────► PostgreSQL
   │
   └──────────────► Python AI Service
                         │
                         ▼
                        LLM
```

During local development, all services can run locally.

---

# 19. Architectural Principles

The project follows these principles:

### Separation of concerns

NestJS owns business logic.

Python owns AI logic.

Next.js owns presentation.

PostgreSQL owns persistent state.

### AI is not the source of truth

The AI can reason and propose actions, but the backend validates and executes them.

### Human-in-the-loop

External side effects require explicit approval.

### Multi-tenant by design

Organization isolation is part of the architecture, not a future feature.

### MVP first

The architecture must allow future expansion without implementing unnecessary complexity in the first version.
