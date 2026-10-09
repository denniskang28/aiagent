# Sales Advisory Agent --- Detailed Agentic AI Design Example

## 1. Purpose

This document provides a concrete reference design for a **Sales
Advisory Agent** in an enterprise Agentic AI Workbench for life
insurance agents.

The example demonstrates how to separate and connect:

-   **Main Agent / Orchestrator**
-   **Specialized Sales Advisory Agent**
-   **Business Skills**
-   **AI Tools / MCP APIs**
-   **Business Application APIs**
-   **Knowledge Bases**
-   **Enterprise Core Systems**

The central design principle is:

> **The Sales Advisory Agent owns the multi-turn advisory conversation,
> reasoning, planning, and local context. Business logic should
> primarily live in Skills and controlled business platforms. Tools
> provide deterministic, governed access to those platforms.**

A second principle is:

> **LLM chooses what should happen next; the runtime determines whether
> it is allowed and how it executes reliably.**

------------------------------------------------------------------------

## 2. Why Sales Advisory Is a Specialized Agent

Not every business capability should be implemented as an Agent.

A simple capability such as `claim-status` can usually be implemented as
a Skill:

``` text
Input
  ↓
Skill
  ↓
Tool/API
  ↓
Output
```

Sales Advisory is different because it can require:

-   independent multi-turn conversation;
-   iterative needs exploration;
-   its own advisory context;
-   planning and replanning;
-   discussion of alternative scenarios;
-   remembering assumptions within the advisory session;
-   reacting to changes such as customer budget;
-   combining several business capabilities over multiple turns.

Example:

``` text
Agent:
"Help me prepare for my meeting with Alice tomorrow."

Sales Advisory Agent:
→ Understand Alice
→ Analyze current coverage
→ Identify possible gaps
→ Recommend discussion topics
→ Prepare suitable solution options

Agent:
"Why do you think her life coverage is insufficient?"

Sales Advisory Agent:
→ Continue the same advisory session
→ Explain the previous analysis

Agent:
"What if her budget is only HKD 3,000 per month?"

Sales Advisory Agent:
→ Update advisory assumptions
→ Re-run affordability analysis
→ Re-evaluate solution options
```

This persistent advisory interaction is the main reason to model Sales
Advisory as a specialized Agent rather than one large Skill.

------------------------------------------------------------------------

## 3. Position in the Overall Architecture

``` text
                         User / Insurance Agent
                                  │
                                  ▼
                     Main Agent / Orchestrator
                                  │
                    Start / Resume / Interrupt
                                  │
                                  ▼
                     Sales Advisory Agent
                                  │
                  Reason / Plan / Converse
                                  │
                 ┌────────────────┼────────────────┐
                 │                │                │
                 ▼                ▼                ▼
              Skills        Direct Read Tools   Knowledge
                 │                │                │
                 └────────────────┼────────────────┘
                                  ▼
                           MCP / AI Tools
                                  │
                                  ▼
                       Business Application APIs
                                  │
          ┌───────────────────────┼────────────────────────┐
          ▼                       ▼                        ▼
 Customer & Sales          Decisioning & NBA        Knowledge & GenAI
 Journey & Work            Product / Policy         Engagement
          │                       │                        │
          └───────────────────────┼────────────────────────┘
                                  ▼
                       Enterprise Core Systems
```

The Main Agent still owns the overall Workbench conversation. The Sales
Advisory Agent owns only the specialized advisory sub-conversation.

------------------------------------------------------------------------

## 4. Example Business Scenario

Alice is a customer:

``` text
Age:                38
Marital status:     Married
Children:           2
Current Life Cover: HKD 1.5M
Current CI Cover:   HKD 500K
Medical:            Existing coverage
Life stage:         Family growth
Recent engagement:  Family protection content
Meeting:            Tomorrow afternoon
```

The insurance agent says:

> "I'm meeting Alice tomorrow afternoon. Help me understand her current
> protection situation and what I should focus on."

The Sales Advisory Agent may dynamically execute:

``` text
Customer Brief
      ↓
Needs Analysis
      ↓
Coverage Gap Analysis
      ↓
Affordability Analysis
      ↓
Next Best Action
      ↓
Product / Solution Recommendation
      ↓
Meeting Preparation
      ↓
Dynamic Meeting Workspace
```

Possible output:

``` text
Alice Meeting Advisory

CUSTOMER SNAPSHOT
38 / Married / 2 Children
Family-growth life stage

CURRENT COVERAGE
Life        HKD 1.5M
CI          HKD 500K
Medical     Existing

POTENTIAL NEEDS
Life Protection      HIGH
Education Funding    MEDIUM
CI Protection        MEDIUM

RECOMMENDED DISCUSSION
1. Family income protection
2. Children's education funding
3. Critical illness protection

SUGGESTED QUESTIONS
...

POTENTIAL SOLUTIONS
...

COMPLIANCE REMINDERS
...
```

The Workbench principle is:

> **Chat is the entry; Workspace is the result.**

------------------------------------------------------------------------

## 5. Sales Advisory Agent Definition

A conceptual Agent specification could look like:

``` yaml
id: sales-advisory-agent
name: Sales Advisory Agent

purpose:
  Assist insurance agents with customer needs discovery,
  protection analysis, solution exploration and
  meeting preparation.

session:
  multiTurn: true
  resumable: true
  interruptible: true

context:
  include:
    - currentCustomer
    - currentOpportunity
    - conversationSummary
    - advisoryState

allowedSkills:
  - customer-brief
  - customer-insight
  - needs-analysis
  - coverage-gap-analysis
  - affordability-analysis
  - life-stage-analysis
  - solution-recommendation
  - product-recommendation
  - product-comparison
  - meeting-preparation
  - question-generation
  - objection-preparation
  - proposal-preparation
  - followup-recommendation
  - compliance-check

allowedDirectTools:
  - get_customer_360
  - get_customer_policies
  - search_knowledge

knowledgeDomains:
  - PRODUCT
  - SALES_ADVISORY
  - COMPLIANCE

limits:
  maxPlanSteps: 8
  maxToolCalls: 15

humanApprovalRequired:
  - submit_proposal
  - send_customer_message
```

The Agent should directly call only a limited set of simple read-only
Tools. Complex business capabilities should normally be invoked through
Skills.

------------------------------------------------------------------------

## 6. Recommended Skill Set

A full Sales Advisory Agent may use approximately 12--15 Skills.

  -----------------------------------------------------------------------
  Skill                               Responsibility
  ----------------------------------- -----------------------------------
  `customer-brief`                    Build an actionable Customer 360
                                      summary

  `customer-insight`                  Identify important customer signals
                                      and insights

  `life-stage-analysis`               Interpret customer life stage

  `needs-analysis`                    Perform structured customer needs
                                      analysis

  `coverage-gap-analysis`             Analyze potential protection gaps

  `affordability-analysis`            Assess affordable solution range

  `solution-recommendation`           Recommend solution strategies

  `product-recommendation`            Identify suitable candidate
                                      products

  `product-comparison`                Compare candidate products

  `meeting-preparation`               Build the meeting advisory
                                      workspace

  `question-generation`               Generate useful discovery questions

  `objection-preparation`             Prepare for likely objections

  `proposal-preparation`              Prepare proposal artifacts/options

  `followup-recommendation`           Recommend next actions

  `compliance-check`                  Check relevant advisory/output
                                      compliance
  -----------------------------------------------------------------------

A practical MVP can begin with ten:

``` text
customer-brief
needs-analysis
coverage-gap-analysis
affordability-analysis
product-recommendation
product-comparison
meeting-preparation
objection-preparation
proposal-preparation
compliance-check
```

------------------------------------------------------------------------

## 7. Skill Example --- Customer Brief

The Agent calls:

``` text
customer-brief(customerId="C10001")
```

The Skill can orchestrate several deterministic Tools:

``` text
Customer Brief Skill
        │
        ├── get_customer_360
        ├── get_customer_policies
        ├── get_customer_interactions
        ├── get_customer_opportunities
        └── get_customer_claims
                 │
                 ▼
          Structured aggregation
                 │
                 ▼
          LLM summarization
                 │
                 ▼
            CustomerBrief
```

Example output contract:

``` json
{
  "customer": {},
  "family": {},
  "financial": {},
  "policies": [],
  "recentInteractions": [],
  "opportunities": [],
  "signals": [],
  "summary": "..."
}
```

The Sales Advisory Agent therefore does **not** need to understand five
separate backend APIs just to obtain a useful customer brief.

------------------------------------------------------------------------

## 8. Tool Example --- get_customer_360

An AI-facing MCP Tool might expose:

``` json
{
  "name": "get_customer_360",
  "description": "Retrieve the authorized 360-degree customer profile.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "customerId": {
        "type": "string"
      }
    },
    "required": ["customerId"]
  }
}
```

Its implementation path could be:

``` text
MCP Tool
get_customer_360
        │
        ▼
Customer 360 API
        │
        ▼
360 Data / Customer & Sales Platform
        │
        ├── CRM
        ├── Policy data
        ├── Interaction data
        ├── Consent
        └── Enterprise Data Platform
```

The MCP Tool is therefore an **AI-safe semantic contract**, not
necessarily a one-to-one copy of an existing REST endpoint.

------------------------------------------------------------------------

## 9. Customer Brief End-to-End Call Chain

``` text
Sales Advisory Agent
        │
        ▼
customer-brief Skill
        │
        ├──────────────────┐
        ▼                  ▼
get_customer_360     get_customer_policies
        │                  │
        ▼                  ▼
Customer 360 API       Policy 360 API
        │                  │
        ▼                  ▼
360 Data Platform      Policy Domain
        │                  │
        ▼                  ▼
CRM / MDM             Policy Admin
```

Additional interaction history can follow:

``` text
get_customer_interactions
        ↓
Interaction API
        ↓
Customer & Sales Platform
        ↓
CRM / LINE / Email / AIA One
```

------------------------------------------------------------------------

## 10. Skill Example --- Needs Analysis

The Agent calls:

``` text
needs-analysis(customerId="C10001")
```

The LLM should **not** independently invent or calculate controlled
insurance needs.

Recommended design:

``` text
                needs-analysis Skill
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
   Customer Data    Financial Data   Existing Policies
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                Decisioning Platform
                         │
                         ▼
                Needs Analysis Engine
                         │
            ┌────────────┼────────────┐
            ▼            ▼            ▼
          Life           CI        Education
           Gap           Gap          Gap
                         │
                         ▼
                   Explanation Data
                         │
                         ▼
                        LLM
                         │
                         ▼
             Human-readable explanation
```

Typical Tools:

``` text
get_customer_360
get_customer_financial_profile
get_customer_policies
run_needs_analysis
calculate_protection_gap
calculate_affordability
```

The critical Tool:

``` text
run_needs_analysis
        ↓
Decisioning API
        ↓
Decisioning & NBA Platform
        ↓
Needs Analysis Engine
        ↓
Rules / Models / Controlled Calculations
```

The boundary is important:

> **Decisioning owns controlled business decisions. The Agent owns
> reasoning and orchestration around those decisions.**

------------------------------------------------------------------------

## 11. Skill Example --- Coverage Gap Analysis

Possible result:

``` text
Current Life Coverage: HKD 1.5M
Estimated Need:        HKD 5.2M
Potential Gap:         HKD 3.7M
```

Execution:

``` text
coverage-gap-analysis
        │
        ├── get_customer_policies
        ├── get_customer_financial_profile
        └── calculate_protection_gap
                         │
                         ▼
                 Decisioning Platform
```

The controlled engine calculates the gap.

The LLM can explain:

-   why the result matters;
-   which assumptions were used;
-   what the agent should discuss with the customer.

The LLM should not be the authoritative calculator.

------------------------------------------------------------------------

## 12. Skill Example --- Product Recommendation

Once a potential need is identified:

``` text
product-recommendation
```

can execute:

``` text
Customer Need
     +
Affordability
     +
Existing Policies
     +
Product Eligibility
     +
Suitability Rules
     │
     ▼
Decisioning Platform
     │
     ▼
Candidate Products
     │
     ▼
Product Knowledge
     │
     ▼
Compliance
     │
     ▼
Recommendation
```

Typical Tools:

``` text
get_product_catalog
get_product_detail
check_product_eligibility
get_next_best_product
search_knowledge
```

The underlying systems serve different purposes:

``` text
Product API
→ What products exist and what are their structured attributes?

Decision API
→ Which products/solutions are potentially suitable?

Knowledge API
→ What do approved product, sales and compliance documents say?
```

These responsibilities should remain separate.

------------------------------------------------------------------------

## 13. Skill Example --- Meeting Preparation

The final meeting preparation Skill can consume structured outputs
generated earlier:

``` text
meeting-preparation
       │
       ├── CustomerBrief
       ├── NeedsAnalysis
       ├── CoverageGap
       ├── ProductRecommendation
       ├── RecentInteractions
       └── NextBestAction
               │
               ▼
         Sales Knowledge
               │
               ▼
         LLM Generation
               │
               ▼
       MeetingWorkspace
```

Example workspace:

``` text
Alice Meeting Workspace

CUSTOMER SNAPSHOT
...

KEY NEEDS
...

POTENTIAL GAPS
...

RECENT ENGAGEMENT
...

RECOMMENDED TOPICS
...

QUESTIONS TO ASK
...

POTENTIAL PRODUCT / SOLUTION OPTIONS
...

POTENTIAL OBJECTIONS
...

COMPLIANCE REMINDERS
...
```

------------------------------------------------------------------------

## 14. Multi-Turn Example --- Customer Budget Changes

After the initial analysis, the user says:

> "What if Alice can spend only HKD 3,000 per month?"

The Sales Advisory Agent already holds task-scoped context:

``` text
AgentSession

customer = Alice
needsAnalysis = NA001
currentGap = ...
candidateSolutions = ...
meeting = tomorrow
```

The continuation can be handled as:

``` text
User
 │
 ▼
Sales Advisory Agent
 │
 ├── Update advisory assumption: budget = HKD 3,000
 │
 ▼
affordability-analysis
 │
 ▼
calculate_affordability
 │
 ▼
Decisioning Platform
 │
 ▼
Affordable solution range
 │
 ▼
product-recommendation
 │
 ▼
Revised candidate solutions
```

There is no need to rebuild the Customer Brief unless relevant customer
data has become stale or the runtime policy requires refresh.

This is a key benefit of the specialized Agent.

------------------------------------------------------------------------

## 15. Interrupt / Resume Example

Suppose Sales Advisory is active:

``` text
activeAgent = sales-advisory-agent
activeAgentSessionId = SA-1001
```

The user suddenly asks:

> "Was Alice's last hospitalization claim paid?"

The Main Orchestrator determines that this is not a continuation of
Sales Advisory.

Recommended flow:

``` text
Sales Advisory Agent
        │
      SUSPEND
        │
        ▼
Main Agent / Orchestrator
        │
        ▼
claim-status Skill
        │
        ▼
get_claim_status
        │
        ▼
Claims API
        │
        ▼
Claims System
```

After answering:

``` text
Alice's hospitalization claim was approved...
Payment status: ...

[ Resume Sales Advisory ]
```

The Orchestrator owns the overall conversation; the Sales Advisory Agent
owns only its sub-conversation.

------------------------------------------------------------------------

## 16. Proposal Preparation

The user continues:

> "Prepare an option for Alice."

The Agent invokes:

``` text
proposal-preparation
```

Possible Tool calls:

``` text
get_customer_360
get_customer_policies
get_product_detail
get_product_rates
calculate_premium
check_eligibility
generate_proposal
```

Underlying application mapping:

``` text
calculate_premium
        ↓
Product / Illustration API
        ↓
Illustration Engine
```

``` text
check_eligibility
        ↓
Decisioning / Product API
        ↓
Eligibility Rules
```

``` text
generate_proposal
        ↓
Document API
        ↓
Proposal Generation Platform
```

------------------------------------------------------------------------

## 17. Human-in-the-Loop for Customer-Facing Actions

Preparing a proposal is different from sending it.

If the Agent decides:

``` text
send_customer_proposal
```

the execution path should be:

``` text
Sales Advisory Agent
        ↓
Requested Action
        ↓
Runtime Policy
        ↓
Risk Classification:
CUSTOMER_FACING
        ↓
Human Approval Required
```

Workbench example:

``` text
Ready to send to Alice

Proposal:
Family Protection Plan

Channel:
LINE

Message:
Hi Alice, based on our discussion...

[ Edit ]

[ Cancel ]                [ Approve & Send ]
```

Only after approval:

``` text
send_customer_message
        ↓
Communication API
        ↓
LINE / Email / Other Channel
```

Core principle:

> **LLM chooses the action; Runtime authorizes the action.**

------------------------------------------------------------------------

## 18. Sales Advisory Agent Skill Map

``` text
                     SALES ADVISORY AGENT
                              │
          ┌───────────────────┼────────────────────┐
          │                   │                    │
          ▼                   ▼                    ▼
      Understand            Analyze              Prepare
          │                   │                    │
          ▼                   ▼                    ▼
 customer-brief        needs-analysis       meeting-preparation
 customer-insight      coverage-gap         question-generation
 life-stage-analysis   affordability        objection-preparation
 customer-timeline     suitability          proposal-preparation
                       product-rec          followup-recommendation
                       product-compare
                              │
                              ▼
                       compliance-check
```

The Agent decides which capability is needed. The Skill owns the bounded
capability. The Tool performs deterministic access/execution.

------------------------------------------------------------------------

## 19. Recommended MCP / AI Tool Map

A practical Sales Advisory Agent may require approximately 20--25 Tools.

  AI Tool / MCP Tool                 Underlying Business API / Module
  ---------------------------------- ----------------------------------
  `get_customer_360`                 Customer 360 API
  `get_customer_financial_profile`   Customer / Data API
  `get_customer_interactions`        Interaction API
  `get_customer_consent`             Consent API
  `get_customer_policies`            Policy 360 API
  `get_policy_detail`                Policy API
  `get_product_catalog`              Product API
  `get_product_detail`               Product API
  `get_product_rates`                Product / Illustration API
  `run_needs_analysis`               Decisioning API
  `calculate_protection_gap`         Decisioning API
  `calculate_affordability`          Decisioning API
  `check_product_eligibility`        Decisioning / Product API
  `get_next_best_product`            NBA API
  `get_next_best_action`             NBA API
  `search_knowledge`                 Knowledge API
  `get_approved_content`             Content API
  `compliance_check`                 Compliance / Decision API
  `calculate_premium`                Illustration API
  `generate_proposal`                Document API
  `create_followup`                  Journey / Work API
  `schedule_meeting`                 Calendar API
  `send_customer_message`            Communication API

------------------------------------------------------------------------

## 20. Knowledge Required by the Agent

The Sales Advisory Agent should not have unrestricted access to every
enterprise Knowledge Base.

Its default Knowledge Policy can include:

### Product Knowledge

-   policy wording;
-   product features;
-   benefits;
-   riders;
-   exclusions;
-   eligibility;
-   product FAQ;
-   approved product materials.

### Sales & Advisory Knowledge

-   needs discovery guidance;
-   sales methodology;
-   conversation guides;
-   objection handling;
-   advisory best practices;
-   meeting preparation guidance.

### Compliance & Regulatory Knowledge

-   suitability;
-   disclosure requirements;
-   customer communication rules;
-   consent requirements;
-   approved sales practices;
-   applicable internal compliance guidance.

Additional Knowledge Domains can be enabled by specific Skills when
required.

Operational data such as Customer 360, Policy, Claim, Application,
Journey State and Agent Performance should **not** be treated as RAG
knowledge. These should be accessed as structured data through
controlled APIs/Tools.

------------------------------------------------------------------------

## 21. Business Application Mapping

``` text
Sales Advisory Agent
        │
      Skills
        │
    MCP / AI Tools
        │
        ▼
┌─────────────────────────────────────────────┐
│             BUSINESS APPLICATIONS           │
│                                             │
│ Customer & Sales Platform                   │
│ ├─ Customer 360 API                         │
│ ├─ Interaction API                          │
│ └─ Opportunity API                          │
│                                             │
│ Policy / Product Platform                   │
│ ├─ Policy API                               │
│ ├─ Product API                              │
│ └─ Illustration API                         │
│                                             │
│ Decisioning & NBA Platform                  │
│ ├─ Needs Analysis API                       │
│ ├─ Protection Gap API                       │
│ ├─ Affordability API                        │
│ ├─ Recommendation API                       │
│ └─ NBA API                                  │
│                                             │
│ Knowledge & Generative AI Platform          │
│ ├─ Knowledge Search API                     │
│ ├─ Product Knowledge                        │
│ ├─ Sales Knowledge                          │
│ └─ Compliance Knowledge                     │
│                                             │
│ Journey & Work Platform                     │
│ ├─ Journey API                              │
│ ├─ Work Item API                            │
│ └─ Approval API                             │
│                                             │
│ Engagement / Communication Platform         │
│ ├─ Content API                              │
│ └─ Communication API                        │
└─────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────┐
│          ENTERPRISE CORE SYSTEMS            │
│                                             │
│ CRM                                         │
│ Policy Administration                       │
│ Product / Illustration                      │
│ Underwriting                                │
│ Claims                                      │
│ LINE / WhatsApp / Email / AIA One           │
│ Calendar                                    │
│ Data Warehouse / Data Platform              │
│ Content Management System                   │
└─────────────────────────────────────────────┘
```

------------------------------------------------------------------------

## 22. Three API Layers Must Remain Separate

A critical implementation rule is to distinguish:

1.  **Skill Contract**
2.  **AI Tool / MCP Contract**
3.  **Business Application API**

Example:

``` text
Sales Advisory Agent
        │
        │ Skill call
        ▼
needs-analysis
────────────────────────────────
AI Business Capability
        │
        │ MCP call
        ▼
run_needs_analysis
────────────────────────────────
AI-safe Semantic Tool Contract
        │
        │ REST / gRPC
        ▼
POST /decision/v1/needs-analysis
────────────────────────────────
Business Application API
        │
        ▼
Needs Analysis Engine
Rules / Models / Calculations
```

Do not expose every existing enterprise REST endpoint directly to the
LLM.

The MCP / AI Tool layer should provide a semantic and governed facade:

``` text
Existing Enterprise API
        │
        ▼
MCP / Tool Adapter
        │
        ├── Semantic schema
        ├── Authentication / Authorization
        ├── Permission enforcement
        ├── Input validation
        ├── PII controls
        ├── Risk classification
        ├── Audit
        ├── Idempotency
        └── Error normalization
        │
        ▼
Skill / Agent Runtime
```

------------------------------------------------------------------------

## 23. Skill Specification Example

A Skill can declare its dependencies rather than forcing the Agent to
understand implementation details.

``` yaml
id: meeting-preparation
name: Meeting Preparation

description:
  Prepare a structured customer meeting advisory workspace.

inputs:
  customerId:
    type: string
    required: true

  meetingId:
    type: string
    required: false

context:
  customer360: true
  advisoryState: true
  conversationSummary: true

knowledge:
  allowedDomains:
    - PRODUCT
    - SALES_ADVISORY
    - COMPLIANCE

tools:
  - get_customer_360
  - get_customer_policies
  - get_customer_interactions
  - get_next_best_action
  - search_knowledge

permissions:
  customerScope: ASSIGNED_CUSTOMER

risk:
  classification: READ_ONLY

output:
  type: MeetingPreparationWorkspace
```

The Agent only needs to see:

``` text
meeting-preparation(customerId, meetingId)
```

It does not need to understand every downstream API.

------------------------------------------------------------------------

## 24. Recommended Runtime Responsibilities

The Agentic Runtime should enforce controls outside the LLM.

### Agent Reasoning Core

Responsible for:

-   understanding the advisory goal;
-   deciding which Skill to use;
-   planning/replanning;
-   interpreting Skill results;
-   maintaining advisory conversation context;
-   composing explanations;
-   deciding when to request an action.

### Deterministic Runtime / Control Plane

Responsible for:

-   identity;
-   authorization;
-   session lifecycle;
-   context filtering;
-   Tool availability;
-   Agent lifecycle;
-   timeout;
-   step limits;
-   approval policy;
-   risk classification;
-   retry;
-   idempotency;
-   audit;
-   traceability;
-   interrupt/resume.

This provides **bounded autonomy**, rather than unrestricted autonomous
execution.

------------------------------------------------------------------------

## 25. End-to-End Alice Execution Trace

A complete example:

``` text
USER
"I'm meeting Alice tomorrow. Help me prepare."

        ↓

MAIN AGENT
Recognizes a complex advisory task.

        ↓

start_agent(
  agentId = "sales-advisory-agent",
  customerId = "C10001"
)

        ↓

SALES ADVISORY AGENT
Goal:
Prepare Alice meeting.

        ↓

customer-brief

        ↓

get_customer_360
get_customer_policies
get_customer_interactions

        ↓

Customer & Sales / Policy APIs

        ↓

CustomerBrief

        ↓

needs-analysis

        ↓

run_needs_analysis

        ↓

Decisioning Platform

        ↓

Potential protection needs identified

        ↓

coverage-gap-analysis

        ↓

calculate_protection_gap

        ↓

Controlled gap result

        ↓

affordability-analysis

        ↓

calculate_affordability

        ↓

Affordable range

        ↓

product-recommendation

        ↓

get_next_best_product
get_product_detail
search_knowledge

        ↓

Candidate solutions + approved evidence

        ↓

compliance-check

        ↓

meeting-preparation

        ↓

DYNAMIC WORKSPACE

Alice Meeting Preparation
├─ Customer Snapshot
├─ Current Policies
├─ Potential Needs
├─ Potential Gaps
├─ Recent Engagement
├─ Recommended Topics
├─ Suggested Questions
├─ Potential Solutions
├─ Objection Preparation
└─ Compliance Reminders
```

Then:

``` text
USER
"What if she can only spend HKD 3,000/month?"

        ↓

SAME SALES ADVISORY AGENT SESSION

        ↓

Update budget assumption

        ↓

affordability-analysis

        ↓

product-recommendation

        ↓

Updated solution options
```

Then:

``` text
USER
"Prepare the proposal."

        ↓

proposal-preparation

        ↓

calculate_premium
check_eligibility
generate_proposal

        ↓

Proposal Draft
```

Then:

``` text
USER
"Send it to Alice."

        ↓

Sales Advisory Agent requests action

        ↓

Runtime Risk Policy

CUSTOMER_FACING

        ↓

HUMAN APPROVAL

        ↓

Approve & Send

        ↓

send_customer_message

        ↓

Communication Platform

        ↓

LINE / Email / Approved Channel
```

------------------------------------------------------------------------

## 26. Recommended MVP Scope

For an initial implementation, avoid building a large number of Agents.

### Agent

``` text
Sales Advisory Agent
```

### Core Skills

``` text
customer-brief
needs-analysis
coverage-gap-analysis
affordability-analysis
product-recommendation
product-comparison
meeting-preparation
objection-preparation
proposal-preparation
compliance-check
```

### Core Tools

``` text
get_customer_360
get_customer_financial_profile
get_customer_interactions
get_customer_consent
get_customer_policies
get_policy_detail

get_product_catalog
get_product_detail
get_product_rates

run_needs_analysis
calculate_protection_gap
calculate_affordability
check_product_eligibility
get_next_best_product
get_next_best_action

search_knowledge
get_approved_content
compliance_check

calculate_premium
generate_proposal

create_followup
schedule_meeting
send_customer_message
```

### Knowledge Domains

``` text
PRODUCT
SALES_ADVISORY
COMPLIANCE
```

This is enough to demonstrate the core Agentic architecture without
overbuilding the first version.

------------------------------------------------------------------------

## 27. Key Architecture Principles Demonstrated by This Example

1.  **Main Agent owns overall orchestration.**
2.  **Sales Advisory Agent owns the specialized multi-turn advisory
    conversation.**
3.  **Skill owns a reusable bounded business capability.**
4.  **Tool owns deterministic access or execution.**
5.  **Business Application owns business services and authoritative
    logic.**
6.  **Decisioning owns controlled business decisions and calculations.**
7.  **Knowledge Platform owns governed knowledge retrieval.**
8.  **Operational data is not a Knowledge Base.**
9.  **MCP is an AI-safe semantic API layer, not a raw mirror of every
    REST endpoint.**
10. **Complex capabilities should normally go through Skills rather than
    exposing many low-level Tools to the Agent.**
11. **Customer-facing and regulated actions require deterministic policy
    enforcement and, where required, human approval.**
12. **The Workbench remains structured business UI: conversational AI
    initiates work, while Dynamic Workspaces present the result.**
13. **The architecture targets bounded autonomy, not maximum autonomy.**

------------------------------------------------------------------------

## 28. Reference Architecture Summary

``` text
                         USER
                           │
                           ▼
                 MAIN AGENT / ORCHESTRATOR
                           │
                           ▼
                   SALES ADVISORY AGENT
                           │
                Reason / Plan / Converse
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
            SKILLS      READ TOOLS   KNOWLEDGE
              │            │            │
              └────────────┼────────────┘
                           ▼
                     MCP / AI TOOLS
                           │
                           ▼
                  BUSINESS APPLICATION APIs
                           │
       ┌───────────────────┼────────────────────┐
       │                   │                    │
       ▼                   ▼                    ▼
Customer & Sales     Decisioning / NBA    Knowledge / GenAI
Policy / Product     Journey & Work       Engagement
       │                   │                    │
       └───────────────────┼────────────────────┘
                           ▼
                  ENTERPRISE CORE SYSTEMS
```

The most important implementation boundary can be summarized as:

> **Agent reasons. Skill encapsulates capability. Tool executes.
> Business Application owns authoritative business behavior. Knowledge
> supplies governed evidence. Runtime controls permission and risk.**
