# Handoff — Agentic AI Insurance Agent Workbench

## 1. Goal

Design an enterprise-grade **Agent AI Assistant / Agent Workbench for life insurance agents and agent leaders**. It is not a generic chatbot. It combines structured business UI, proactive recommendations, journey/work management, decisioning, knowledge, and bounded Agentic AI execution.

Target product principle:

> **Structured Business UI + Agentic AI Interaction + Proactive Recommendations + Human-in-the-Loop Control**

The local BU solution should follow the Group blueprint, use AIACN as reference rather than a mandatory copy, and localize products, channels, data, regulations, consent, workflows and integrations.

---

## 2. Original 9 Logical Subsystems

1. **AI Workbench & Experience** — Home/Today, Morning Brief, AI Assistant, recommendations, nudges, tasks, approvals, Customer/Agent workspaces, explainability, multi-channel experience.
2. **Customer & Sales Platform** — Customer 360, leads/opportunities, engagement, meetings, follow-up, needs analysis, proposal, application/UW, service/claims, customer timeline.
3. **Agent Growth & Coaching Platform** — goals, success plan, performance, capability assessment, coaching, role play, learning, improvement tracking.
4. **Leader & Recruitment Platform** — candidate/recruitment, team dashboard, leader coaching, agent risk, recruitment funnel, team NBA.
5. **Journey & Work Orchestration Platform** — journey definition/instance, stage, transitions, triggers, milestones, work items, timers/SLA, approval, escalation, retry, audit.
6. **Decisioning & NBA Platform** — ranking, scoring, NBA, Next Best Customer/Content/Product, needs/affordability/coverage analysis, recommendations, explainability, nudge prioritization.
7. **Knowledge & Generative AI Platform** — ingestion/RAG/reranking/citation, AI experts, conversation intelligence, content generation, document generation.
8. **360 Data & Semantic Platform** — customer/agent/candidate/team 360, policy/claim/application/interaction views, features, identity, consent, lineage, semantics.
9. **Integration & AI Control Platform** — enterprise integrations/tools, SSO/authz, model gateway, guardrails, compliance, audit, observability.

These are **logical capability boundaries, not necessarily 9 microservices**.

---

## 3. UI Ownership Decision

Customer & Sales has its own UI modules, but **not a separate end-user application**.

Users see unified products:

- **Agent Workbench** — mobile-first + web
- **Leader Workbench** — web-first + mobile-lite

Example Agent Workbench shell:

```text
Home / Today
Customers
AI
Tasks
Me
```

Customer & Sales owns pages/modules inside the Workbench such as Customer List, Customer 360, Engagement, Meeting Brief, Needs Analysis, Proposal, Application/UW, Service/Claims.

> **Domain UI ownership != separate app.**

---

## 4. Personalized Customer Engagement

This is now a formal core capability of Customer & Sales.

Flow:

```text
Customer Understanding
→ AI recommends suitable content
→ Explain why
→ Agent previews content
→ AI prepares engagement/message
→ Agent edits / approves
→ Select channel + consent check
→ Send
→ Track delivered/open/click/reply
→ Detect customer interest
→ Recommend follow-up
```

Example Alice:

- 38, married, 2 children, family-growth stage
- education savings gap
- recent family-protection engagement
- AI recommends Education Planning content
- agent approves LINE message
- customer clicks Learn More
- system raises interest signal
- AI recommends calling today

This spans:

- #2 Customer & Sales — engagement record, customer timeline
- #6 Decisioning — what content / when / why
- #7 Knowledge/GenAI — content + personalized wording
- #8 360 Data — profile, life stage, policy, consent
- #9 Integration — LINE/WhatsApp/AIA One/Email delivery
- #1 Workbench — preview/edit/approve/send/follow-up UX

---

## 5. Journey vs Decision — Final Boundary

### Journey

Answers:

> **Where are we in the business process and what happens next?**

Owns:

- authoritative business process state
- stage / transition
- work item
- timer / SLA
- approval
- retry / escalation
- long-running process lifecycle

Example Customer Nurturing:

```text
TARGET → NURTURE → INTERESTED → CONTACTED → APPOINTMENT
```

### Decision

Answers:

> **Given the current facts, what is the best business choice?**

Owns:

- ranking / scoring
- customer priority
- content recommendation
- needs analysis
- interest assessment
- NBA
- product/solution recommendation

Example:

```text
customer.content.clicked
        ↓
Journey: current stage = NURTURE
        ↓
Decision: Interest = HIGH
        ↓
Journey: NURTURE → INTERESTED
        ↓
Decision: Next Best Action = CALL
        ↓
Journey: create follow-up work item
```

Key principle:

> **Decision makes intelligent choices; Journey turns those choices into managed business execution.**

---

## 6. Evolution to Agentic AI

Agentic AI does **not replace** the 9-subsystem architecture.

Instead:

> **9-system capability architecture + Agentic Orchestration Layer**

Original architecture answers:

> What capabilities exist?

Agentic architecture answers:

> How can AI dynamically select and compose those capabilities at runtime?

---

## 7. Main Agent vs Domain Agents — Final Direction

Default recommendation changed from many Domain Agents to:

```text
Main Agent / Orchestrator
        ↓
Domain Skills
        ↓
Business Tools
```

Principle:

> **Skill first. Agent only when necessary.**

Domain boundaries still exist as namespaces, ownership, permissions and context policy, for example:

```text
skills.customer.*
skills.coaching.*
skills.recruitment.*
```

But:

> **Domain boundary != Agent boundary.**

---

## 8. When a Specialized Agent Is Needed

Use a Sub-Agent only when the capability genuinely needs one or more of:

1. Independent multi-turn conversation
2. Its own planning/reasoning loop
3. Dedicated local context/memory
4. Distinct context policy
5. Strong permission boundary
6. External Agent runtime

Good examples:

- AI Role Play Agent
- Deep Coaching Agent
- External Copilot Studio Agent
- Dify Agent
- LangGraph Agent

Most MVP use cases should remain Skills.

---

## 9. Hybrid Orchestrator

Preferred architecture:

> **Deterministic Control Plane + Agentic Reasoning Core**

```text
User / Event
    ↓
Orchestrator Control Plane
    ├─ Session
    ├─ Auth / Permission
    ├─ Agent lifecycle
    ├─ Interrupt / Resume
    ├─ Approval policy
    ├─ Timeout / Step limits
    ├─ Audit / Trace
    ↓
Agentic Reasoning Core
    ├─ Goal understanding
    ├─ Continuation detection
    ├─ Skill / Agent routing
    ├─ Planning / Replanning
    ├─ Tool calling
    └─ Response composition
    ↓
Skills / Specialized Agents / Actions
    ↓
Tools
    ↓
Journey / Decision / Knowledge / 360 / Integration
```

Core rule:

> **LLM decides what should happen next. Runtime decides whether it is allowed and how it executes reliably.**

---

## 10. Tool-Calling Orchestrator

The Main LLM Agent should be implemented as a **Tool-Calling Agent**.

It can choose to:

- call a Skill
- start a specialized Agent
- continue/resume an Agent session
- invoke a business action
- run a multi-step Plan

Example:

User:

> Prepare Alice’s meeting this afternoon. If she has a material protection gap, also prepare proposal options.

Possible execution:

```text
customer_brief
    ↓
needs_analysis
    ↓
if material_gap == true
    → proposal_preparation
    ↓
prepare_meeting
```

The Main Agent should **not** directly see low-level URLs, SQL, Kafka, CRM implementation details, etc.

---

## 11. Skill / Tool / Agent Definitions

### Skill

A reusable bounded business capability.

Characteristics:

- stateless or task-scoped
- structured input/output
- no independent long-term conversation

Examples:

- customer-brief
- meeting-preparation
- needs-analysis
- draft-customer-message
- claim-status
- application-followup
- performance-diagnosis

### Tool

Actual deterministic execution capability.

Examples:

- getCustomer360
- getPolicies
- getClaimStatus
- searchKnowledge
- createFollowUp
- sendCustomerMessage
- scheduleMeeting

### Agent

Owns an independent multi-turn sub-conversation.

Examples:

- roleplay-agent
- coaching-agent
- external-copilot-agent

---

## 12. Agent Lifecycle Tools

Use generic Agent lifecycle tools, not separate tools per Agent:

```text
start_agent
send_agent_turn
resume_agent
end_agent
```

Example:

```json
{
  "agentId": "roleplay-agent",
  "goal": "Practice affordability objection",
  "context": {
    "productId": "CI-101"
  }
}
```

Runtime returns an `agentSessionId` and status such as `WAITING_USER`.

---

## 13. Active Agent / Interrupt / Resume

The Orchestrator owns the overall conversation.

If a specialized Agent is active:

```text
activeAgent = roleplay-agent
activeAgentSessionId = RA-1001
```

New turns normally go to that Agent.

But the Orchestrator must detect interruptions.

Example:

User is in Role Play, then asks:

> What is Alice’s claim status?

System should:

1. detect non-continuation
2. suspend Role Play
3. execute claim-status Skill
4. return answer
5. show **Resume Role Play**

Final ownership model:

> **Orchestrator owns the conversation.**  
> **Agent owns a sub-conversation.**  
> **Skill owns a capability.**  
> **Tool owns an action.**

---

## 14. Session / Context Data Model

Recommended core objects:

### ConversationSession

```text
sessionId
userId
activeAgentId
activeAgentSessionId
status
summary
structuredState
```

### AgentSession

```text
agentSessionId
agentId
parentSessionId
status
localContext
summary
startedAt
lastActiveAt
```

### TaskExecution

```text
executionId
sessionId
type = SKILL | AGENT | PLAN
targetId
status
input
output
approvalState
```

### Handoff

```text
handoffId
from
to
goal
context
summary
result
```

Important:

> Conversation storage may be unified; model context must be filtered by Context Policy.

Do not pass the whole global history to every Sub-Agent.

---

## 15. Routing Strategy

Recommended order:

```text
Incoming Message
      ↓
1. Active Agent Continuation Check
      ↓
2. Explicit UI / Page Context
      ↓
3. Deterministic Rules
      ↓
4. Candidate Skill / Agent Retrieval
      ↓
5. LLM Router
      ↓
Skill / Agent / Plan
```

Router output should be structured, for example:

```json
{
  "executionType": "AGENT",
  "target": "roleplay-agent",
  "mode": "START_NEW_SESSION",
  "confidence": 0.95
}
```

or:

```json
{
  "executionType": "SKILL",
  "target": "claim-status",
  "confidence": 0.98
}
```

---

## 16. Dynamic Tool Set

Do not expose all Skills/Agents to the Main LLM at once.

If there are 100+ Skills:

```text
User Query
    ↓
Candidate Retrieval
    ↓
Top-K Skills / Agents
    ↓
Main LLM chooses among those candidates
```

Example for “Prepare Alice meeting”:

- customer-brief
- meeting-preparation
- needs-analysis
- proposal-preparation

Register these as concrete tools for that turn.

This is preferred over a generic `execute_skill(skillId, input)` because concrete tools give stronger schema and better tool selection.

---

## 17. Tool Categories

### Read Tools

- getCustomer360
- getPolicies
- getClaimStatus
- searchKnowledge

Usually auto-executable.

### Decision Tools

- runNeedsAnalysis
- getNextBestAction
- assessCustomerInterest
- rankCustomers

Call the controlled Decisioning Platform.

### Write / Action Tools

- createFollowUp
- updateCRM
- sendCustomerMessage
- scheduleMeeting

Potentially require approval.

### Agent Lifecycle Tools

- start_agent
- send_agent_turn
- resume_agent
- end_agent

---

## 18. Human-in-the-Loop

Actions should have risk classes:

```text
READ_ONLY
LOW_RISK_WRITE
CUSTOMER_FACING
REGULATED_ACTION
```

Examples:

```text
getCustomer360
→ READ_ONLY
→ auto

createInternalTask
→ LOW_RISK_WRITE
→ usually auto

sendCustomerMessage
→ CUSTOMER_FACING
→ approval

submitProposal
→ REGULATED_ACTION
→ strong approval
```

Key principle:

> **LLM chooses the action; Runtime authorizes the action.**

---

## 19. Execution Limits

Recommended hard limits:

```text
maxPlanSteps = 8
maxToolCalls = 15–20
maxAgentStarts = 2
maxAgentHandoffs = 3
maxAgentDepth = 2
timeout = configurable
```

Do not allow free recursive Agent-to-Agent conversations.

Cross-Agent handoff should always return through the Orchestrator.

---

## 20. Workbench UX Evolution

The visual difference between pre-Agentic and Agentic Workbench should **not** be huge.

Roughly 70–80% of structured business UI can remain.

### Pre-Agentic

- user finds a feature
- user navigates pages
- fixed flows
- chatbot answers
- user manually connects steps

### Agentic

- user states desired outcome
- AI creates a plan
- AI dynamically composes capabilities
- AI executes multi-step work
- user intervenes only at key decisions
- AI produces structured Workspaces
- Human + AI share a work queue

---

## 21. Agentic Workbench UX Additions

Incrementally add to the existing UI:

1. Goal Composer
2. AI Suggested Actions
3. AI Plan
4. Execution Progress
5. Dynamic AI Workspace
6. Human + AI Task Queue
7. Approval Center
8. Interrupt / Resume
9. AI Activity Timeline
10. Explainability
11. Global AI entry
12. Customer Interest Signals
13. Personalized Engagement Plan

Do **not** turn the product into chat-only UI.

---

## 22. Recommended Agent Mobile Navigation

For enterprise adoption, retain the familiar structure:

```text
Home / Today
Customers
AI
Tasks
Me
```

AI should be globally accessible, not only inside the AI tab.

A more aggressive V2 concept was also explored:

```text
Today
Work
People
Me
```

with a floating global AI button.

Current preference:

> Keep existing navigation, but progressively reduce navigation dependency through AI.

---

## 23. Core Agentic UI Patterns

### Goal Composer

```text
What do you want to get done today?
[ Prepare me for Alice’s meeting ]
```

### AI Plan

```text
Goal: Prepare Alice Meeting

AI will:
✓ Review customer profile
✓ Review policies
✓ Analyze interactions
✓ Run needs analysis
✓ Get NBA
✓ Retrieve approved product knowledge
✓ Prepare meeting workspace
```

### Execution Progress

```text
Preparing Alice Meeting

✓ Customer profile loaded
✓ Policies reviewed
✓ Interactions analyzed
✓ Coverage gaps calculated
● Retrieving product knowledge
○ Preparing talking points
```

### Dynamic AI Workspace

Chat is the entry; Workspace is the result.

Example:

```text
Alice Meeting Preparation

Customer Snapshot
Policies
Potential Gaps
Recent Engagement
Recommended Topics
Suggested Questions
Potential Objections

[Practice Role Play]
[Analyze Needs]
[Prepare Proposal]
[Create Follow-up]
```

---

## 24. Human + AI Work Queue

Tasks should evolve to:

```text
MY TASKS
Call Alice

AI TASKS
Preparing David meeting

WAITING FOR ME
Approve Alice message

WAITING FOR OTHERS
Michael income document
```

The UI should always make ownership clear: Human / AI / Customer / Underwriting / System.

---

## 25. Specialized Role Play Agent UX

Role Play is the clearest first specialized multi-turn Agent.

Example:

```text
Scenario: 45-year-old parent
Product: Critical Illness
Persona: Price-sensitive

AI Customer:
“I already have insurance. Why should I pay another $300 per month?”
```

After multiple turns:

```text
Role Play Result
Empathy: 82
Product Knowledge: 91
Objection Handling: 74
Closing: 68

[Practice Again]
[View Coaching Advice]
```

Supports interrupt / resume.

---

## 26. Latest Layered Architecture Direction

### Layer 1 — User Experience

- Agent Workbench Mobile
- Agent Workbench Web
- Leader Workbench
- External messaging/customer channels

### Layer 2 — Interaction & Experience Services

- Conversation Gateway
- Goal / Task UX
- Approval / Explainability
- Notification / Signal Center
- Work queue
- Dynamic Workspace rendering

### Layer 3 — Agentic Orchestration

**Main Agent / Orchestrator**

Deterministic Control Plane:

- auth / permission
- session management
- agent lifecycle
- interrupt / resume
- approval policy
- timeout / step limits
- audit / tracing

Agentic Reasoning Core:

- goal understanding
- continuation detection
- skill / agent routing
- planning / replanning
- tool calling
- response composition

Also:

- Skill Registry
- Agent Registry
- Tool Registry
- Context Policy
- Permission Boundary

### Layer 4 — Execution

Business Skills:

- Customer Brief
- Meeting Preparation
- Needs Analysis
- Proposal Preparation
- Application Follow-up
- Claims Assistant
- Customer Engagement
- Performance Diagnosis

Specialized Multi-Turn Agents:

- Role Play Agent
- Coaching Agent
- External Agent Adapter

### Layer 5 — Core Business / AI Platforms

- Customer & Sales
- Journey
- Decisioning
- Knowledge & Content
- Engagement / Communication

### Layer 6 — Shared Runtime / Data

- LLM / Agent Runtime
- Data & Retrieval
- Event / Workflow
- Observability / Governance

### Layer 7 — Enterprise Systems

- CRM
- Policy Admin
- Underwriting
- Claims
- Product / Content CMS
- LMS
- Calendar / Email
- Identity / SSO
- Messaging Channels
- Data Warehouse

---

## 27. Stable Architecture Principles

1. **Journey owns state.**
2. **Decision owns controlled business decisions.**
3. **Main Agent owns reasoning and orchestration.**
4. **Specialized Agent owns a sub-conversation.**
5. **Skill owns a reusable bounded capability.**
6. **Tool owns deterministic execution.**
7. **LLM chooses; Runtime authorizes.**
8. **Skill first, Agent only when necessary.**
9. **Structured UI remains; Agentic AI reduces navigation burden.**
10. **Bounded autonomy, not maximum autonomy.**

---

## 28. Recommended Next Topics

Continue from this baseline with one of:

1. Detailed Orchestrator technical design
   - interfaces
   - state machine
   - tool loop
   - approval / retry / timeout
   - session store

2. Java / Spring Boot implementation
   - MainAgentRuntime
   - Orchestrator
   - ToolRegistry
   - SkillRuntime
   - AgentRuntime
   - AgentAdapter

3. Skill specification format
   - YAML / JSON metadata
   - input/output schema
   - tool dependencies
   - context policy
   - permission policy

4. Agent specification format
   - session lifecycle
   - handoff context
   - resume policy
   - allowed skills/tools

5. Dynamic tool registration / Top-K candidate routing

6. Dynamic AI Workspace artifact contract and UI renderer

7. Refine the latest architecture diagram so it maps exactly to the original 9 logical subsystems plus the Agentic Orchestration Layer

8. Define MVP flows and delivery roadmap

Recommended first end-to-end MVP flows:

- Morning Brief
- Customer & Policy Brief
- Personalized Customer Engagement
- Meeting Notes & Follow-up
- Role Play Agent as first true multi-turn specialized Agent

---

## 29. Suggested Prompt for a New Chat

Paste this together with this handoff:

> We are continuing the design of an enterprise Agentic AI Workbench for life insurance agents. Treat the attached handoff as the current baseline and do not restart the design from scratch.
>
> Key direction:
> - the original 9 logical capability subsystems remain the foundation
> - add an Agentic Orchestration Layer
> - Main LLM Agent acts as Orchestrator through tool calling
> - prefer Skills over Domain Agents
> - use specialized Sub-Agents only for true independent multi-turn scenarios
> - Orchestrator = deterministic control plane + agentic reasoning core
> - Journey owns business state
> - Decision owns controlled business decisions
> - Main Agent owns reasoning/orchestration
> - Skill owns bounded capability
> - Tool owns deterministic action
> - customer-facing / regulated actions require human-in-the-loop
> - Workbench remains structured UI, enhanced with Goal Composer, AI Plan, AI Progress, Dynamic Workspace, Human+AI Task Queue, Approval, Explainability and Personalized Customer Engagement
>
> Continue from this baseline.
