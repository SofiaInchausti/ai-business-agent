# AI Business Operations Agent

## 1. Product Overview

AI Business Operations Agent is a SaaS platform designed for small and medium-sized businesses.

The platform allows a company to centralize customer information, internal documents, business policies and operational procedures, and interact with an AI agent that can use this information to answer questions and propose or execute business actions.

The agent must understand not only company data, but also the company's defined policies and procedures.

The system is designed around a human-in-the-loop model:

* Reading and retrieving information does not require approval.
* Creating, modifying or executing business actions requires human approval.

The initial MVP will focus on customer management, company knowledge, business policies and email-based actions.

---

## 2. Problem

Small and medium-sized businesses often have important information distributed across different places:

* Customer records
* Internal documents
* Business policies
* Operational procedures
* Email
* Employee knowledge

Employees must manually search this information and determine what action should be taken.

This creates repetitive work and increases the risk of inconsistent decisions or actions that do not follow company procedures.

The product aims to provide an AI assistant that can understand the available information, apply company policies and help employees execute operational tasks.

---

## 3. Target Users

### Primary user

Employees of small and medium-sized businesses who perform operational, administrative, sales or customer-management tasks.

### Administrator

A company administrator who manages:

* Company users
* Customer information
* Company documents
* Business policies
* Agent configuration
* Pending approvals
* Execution history

---

## 4. Core Use Cases

### Use Case 1 — Find inactive customers

The user can ask:

> "Show me customers who have not made a purchase in the last 90 days."

The agent should:

1. Understand the request.
2. Determine that customer data is required.
3. Use the appropriate customer search tool.
4. Identify matching customers.
5. Return the results.

No approval is required because this is a read-only operation.

---

### Use Case 2 — Identify customers with outstanding debt

The user can ask:

> "Which customers currently have overdue payments?"

The agent should:

1. Search customer information.
2. Identify customers with outstanding or overdue balances.
3. Return relevant information.
4. If appropriate, consult the company's collection policy.

No approval is required for retrieving this information.

---

### Use Case 3 — Apply a company policy

The company may have a policy such as:

> Customers with payments overdue by more than 15 days should receive an email reminder.

The user can ask:

> "What should we do with customers who are more than 15 days overdue?"

The agent should:

1. Identify the relevant customers or situation.
2. Retrieve the applicable company policy.
3. Determine the recommended next step.
4. Explain the recommendation and reference the policy used.

The agent must not execute the action automatically.

---

### Use Case 4 — Generate an email

The user can ask:

> "Prepare an email for these customers explaining that their payment is overdue."

The agent should:

1. Identify the relevant customers.
2. Retrieve applicable company policies.
3. Generate personalized email drafts.
4. Present the drafts to the user.
5. Wait for approval.

Creating an email draft is considered a write/action operation and requires approval before execution.

---

### Use Case 5 — Send an email

After the user approves the proposed emails, the system can send them through the configured email provider.

The system must record:

* User who approved the action
* Action performed
* Customers affected
* Date and time
* Result
* Relevant policy or reasoning context

---

### Use Case 6 — Retrieve customer information

The user can ask:

> "What do we know about Juan Pérez?"

The agent should retrieve relevant available information about the customer, such as:

* Customer status
* Last purchase
* Purchase history
* Outstanding balance
* Last interaction
* Relevant notes or information available in the system

The response should focus on information that is relevant and useful to the user's request.

---

## 5. MVP Features

### 5.1 Authentication

Users must be able to:

* Register
* Log in
* Log out
* Maintain an authenticated session

---

### 5.2 Organizations

The application must support multiple companies.

Each organization must have isolated data.

An organization can have multiple users.

Initial roles:

* Admin
* Member

---

### 5.3 Customer Management

The system must allow authorized users to:

* Create customers
* View customers
* Search customers
* View customer details
* Record relevant customer information
* Record purchase information
* Record outstanding balances

The initial customer model should remain intentionally simple.

---

### 5.4 Company Knowledge Base

Authorized users can upload company documents.

Initial supported format:

* PDF

The system must:

1. Receive the document.
2. Extract its text.
3. Split the text into chunks.
4. Generate embeddings.
5. Store the embeddings.
6. Make the information available for semantic search.

The AI agent must be able to retrieve relevant information from the knowledge base.

---

### 5.5 Business Policies

Company policies and procedures will initially be represented through the company knowledge base.

Examples:

* Collection policies
* Customer communication procedures
* Sales policies
* Internal operational procedures

The agent must be able to retrieve relevant policies when answering questions or proposing actions.

---

### 5.6 AI Agent

The system must provide an AI agent capable of:

* Understanding natural language requests
* Maintaining conversation context
* Determining when information retrieval is necessary
* Selecting appropriate tools
* Using customer information
* Searching the company knowledge base
* Applying relevant company policies
* Generating responses
* Proposing actions

The agent must not have unrestricted access to all tools.

Its available tools must be explicitly defined.

---

### 5.7 Agent Tools

Initial tools:

```text
search_customers()
get_customer()
search_knowledge_base()
create_email_draft()
send_email()
```

The agent must select tools based on the user's request.

Tool calls must be validated before execution.

---

### 5.8 Human Approval

Actions that modify data or communicate externally require human approval.

#### Read operations

No approval required.

Examples:

```text
search_customers()
get_customer()
search_knowledge_base()
```

#### Action operations

Approval required.

Examples:

```text
create_email_draft()
send_email()
```

The approval interface must clearly show:

* Proposed action
* Target
* Reason
* Relevant policy when applicable
* Generated content
* Expected result

The user must be able to:

* Approve
* Reject

---

### 5.9 Email Integration

The MVP will support email as the first external action.

The system must be able to:

* Generate email drafts
* Display drafts for approval
* Send approved emails
* Report success or failure

The specific email provider will be defined during the technical architecture phase.

---

### 5.10 Execution History / Audit Log

The system must record important agent actions.

An audit record should contain, when applicable:

* User
* Organization
* Request
* Agent action
* Tool used
* Action status
* Approval status
* Timestamp
* Result
* Error information

Users with the appropriate permissions can view the execution history.

---

## 6. Human-in-the-Loop Rules

The following rule is fundamental to the product:

> The AI agent may read information automatically, but it must not perform external or modifying actions without human approval.

Examples:

| Operation                | Approval |
| ------------------------ | -------- |
| Search customers         | No       |
| View customer            | No       |
| Search company documents | No       |
| Retrieve policies        | No       |
| Generate recommendation  | No       |
| Create email draft       | Yes      |
| Modify customer data     | Yes      |
| Send email               | Yes      |

This rule must be enforced by the backend and must not depend exclusively on the frontend.

---

## 7. Agent Behavior

The agent should follow this general process:

```text
User request
    ↓
Understand intent
    ↓
Determine required information
    ↓
Select appropriate tools
    ↓
Retrieve data
    ↓
Retrieve relevant policies
    ↓
Generate answer or proposed action
    ↓
If action requires approval
    ↓
Request human approval
    ↓
Execute approved action
    ↓
Record execution
```

The agent should not invent company policies or customer information.

When relevant information cannot be found, the agent should clearly state that the information is unavailable.

---

## 8. Multi-Tenant Requirements

The product is intended to operate as a SaaS.

Each organization must have isolated:

* Users
* Customers
* Documents
* Policies
* Conversations
* Agent executions
* Audit logs

A user must never be able to access another organization's data.

---

## 9. MVP Success Criteria

The MVP will be considered successful when a company can complete the following workflow:

```text
1. Create an organization
2. Create or import customers
3. Upload a company policy
4. Ask the AI about customers
5. Ask the AI to identify a business situation
6. Have the AI retrieve the relevant policy
7. Have the AI propose an action
8. Review the proposed action
9. Approve the action
10. Execute the action
11. View the execution in the audit history
```

A complete demonstration should be possible without manual intervention from the development team.

---

## 10. Explicit MVP Constraints

The MVP will intentionally NOT include:

* Multiple external integrations
* WhatsApp integration
* Google Calendar
* Complex CRM functionality
* Advanced workflow builder
* Autonomous actions without approval
* Complex agent memory
* Multiple specialized agents
* Mobile application

These features may be considered for future versions.

---

## 11. Future Possibilities

Potential future features include:

* WhatsApp integration
* Google Calendar integration
* Slack integration
* Multiple specialized agents
* Visual workflow builder
* Structured business rules
* Advanced agent memory
* Automatic actions with configurable permissions
* Agent evaluation dashboard
* Advanced analytics
* More external integrations
