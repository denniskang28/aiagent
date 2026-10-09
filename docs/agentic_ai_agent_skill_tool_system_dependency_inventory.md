# Agentic AI Insurance Workbench --- Agent / Skill / Tool / System Dependency Inventory

## 1. Purpose

This document defines the proposed implementation inventory for the
enterprise **Agentic AI Insurance Agent Workbench**.

It focuses on four questions:

1.  Which **Agents** should be created?
2.  Which reusable **Skills** should be created?
3.  Which **AI Tools / MCP Tools** should be exposed?
4.  Which **Business Applications and enterprise systems** are required
    underneath those Tools?

The architecture follows these principles:

> **Main Agent owns overall reasoning and orchestration.**

> **Specialized Agent owns an independent multi-turn sub-conversation.**

> **Skill owns a reusable bounded business capability.**

> **Tool owns deterministic access or execution.**

> **Business Application owns authoritative business behavior and
> business state.**

> **LLM chooses; Runtime authorizes.**

> **Skill first; Agent only when necessary.**

------------------------------------------------------------------------

# 2. Target Capability Inventory

Indicative target scale:

  Capability                                     MVP   Target State
  ----------------------------------------- -------- --------------
  Main Agent                                       1              1
  Specialized Agents                            1--2           5--8
  Business Skills                             15--20         40--60
  MCP / AI Tools                              25--40        60--100
  Knowledge Domains                             6--8         10--12
  Major Business Application Dependencies       6--8         10--15

The target should not be interpreted as a requirement to create dozens
of autonomous Agents. Most business capabilities should remain Skills.

------------------------------------------------------------------------

# 3. Overall Dependency Model

``` text
                           USERS
                             │
                             ▼
                 MAIN AGENT / ORCHESTRATOR
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
            SKILLS      SPECIALIZED       PLAN
                           AGENTS
              │              │
              └──────────────┼──────────────┐
                             ▼              │
                        MCP / AI TOOLS ◄────┘
                             │
                             ▼
                  BUSINESS APPLICATION APIs
                             │
        ┌────────────────────┼─────────────────────┐
        ▼                    ▼                     ▼
 Customer & Sales      Decisioning / NBA      Knowledge / GenAI
 Journey & Work        Policy / Product       Engagement
 Coaching              Recruitment            Data / Semantic
        │                    │                     │
        └────────────────────┼─────────────────────┘
                             ▼
                   ENTERPRISE CORE SYSTEMS
```

------------------------------------------------------------------------

# 4. Agents to Create

## A01 --- Main Agent / Workbench Orchestrator

### Purpose

The primary AI entry point for Agent Workbench and Leader Workbench.

### Responsibilities

-   understand user goal;
-   detect conversation continuation;
-   detect active specialized Agent;
-   retrieve candidate Skills / Agents;
-   choose Skill / Agent;
-   create execution plan;
-   execute/replan;
-   manage interrupt/resume;
-   request approvals;
-   compose final responses;
-   create Dynamic Workspace output.

### Primary Skills

Potentially all registered Skills, but only a Top-K candidate set should
be exposed for each turn.

### Direct Tools

Only a small set of generic/read-only runtime Tools should be directly
available where appropriate:

``` text
get_current_user_context
get_current_page_context
get_agent_status
start_agent
send_agent_turn
resume_agent
suspend_agent
end_agent
```

### Dependencies

``` text
Agentic Orchestration Platform
Identity / SSO
Permission Service
Session Store
Skill Registry
Agent Registry
Tool Registry
Context Policy
Approval Policy
Audit / Observability
LLM Gateway
```

------------------------------------------------------------------------

## A02 --- Sales Advisory Agent

### Purpose

Multi-turn customer advisory, needs exploration, solution discussion and
meeting preparation.

### Skills

``` text
customer-brief
customer-insight
life-stage-analysis
needs-analysis
coverage-gap-analysis
affordability-analysis
solution-recommendation
product-recommendation
product-comparison
meeting-preparation
question-generation
objection-preparation
proposal-preparation
followup-recommendation
compliance-check
```

### Key Tools

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

### Business Application Dependencies

``` text
Customer & Sales Platform
360 Data & Semantic Platform
Policy Platform
Product Platform
Illustration Engine
Decisioning & NBA Platform
Knowledge & GenAI Platform
Journey & Work Platform
Engagement / Communication Platform
Calendar
```

------------------------------------------------------------------------

## A03 --- Role Play Agent

### Purpose

Independent multi-turn sales/customer simulation.

### Skills

``` text
roleplay-scenario-preparation
customer-persona-generation
objection-generation
roleplay-evaluation
conversation-scoring
coaching-feedback
practice-recommendation
product-explanation
```

### Tools

``` text
get_product_detail
search_knowledge
get_roleplay_scenario
get_competency_model
save_roleplay_session
save_roleplay_score
create_learning_assignment
```

### Dependencies

``` text
Knowledge & GenAI Platform
Product Platform
Agent Growth & Coaching Platform
Learning / LMS
Performance Data Platform
```

------------------------------------------------------------------------

## A04 --- Deep Coaching Agent

### Purpose

Persistent multi-turn coaching for insurance agents.

### Skills

``` text
performance-summary
performance-diagnosis
goal-progress-analysis
activity-analysis
conversion-analysis
pipeline-analysis
capability-assessment
coaching-recommendation
learning-recommendation
practice-recommendation
improvement-plan
```

### Tools

``` text
get_agent_360
get_agent_performance
get_agent_goals
get_sales_activity
get_conversion_metrics
get_pipeline_metrics
get_competency_profile
get_learning_history
search_knowledge
create_improvement_plan
create_learning_assignment
create_coaching_task
```

### Dependencies

``` text
Agent Growth & Coaching Platform
360 Data & Semantic Platform
Customer & Sales Platform
Knowledge Platform
LMS
Journey & Work Platform
```

------------------------------------------------------------------------

## A05 --- Recruitment / Leader Coaching Agent

### Purpose

Multi-turn recruitment and leader development scenarios.

### Skills

``` text
candidate-summary
candidate-assessment
candidate-ranking
recruitment-pipeline-analysis
recruitment-followup
team-performance
agent-risk-detection
leader-coaching
team-nba
```

### Tools

``` text
get_candidate_360
get_candidate_profile
get_recruitment_pipeline
get_team_360
get_team_performance
get_agent_risk_scores
get_team_nba
search_knowledge
create_recruitment_followup
create_leader_task
```

### Dependencies

``` text
Leader & Recruitment Platform
Agent Growth & Coaching Platform
360 Data Platform
Decisioning & NBA Platform
Knowledge Platform
Journey & Work Platform
```

------------------------------------------------------------------------

## A06 --- Knowledge Expert Agent

### Purpose

Complex evidence-based research requiring decomposition and synthesis
across multiple governed knowledge domains.

### Skills

``` text
knowledge-answer
knowledge-synthesis
product-explanation
underwriting-guideline-analysis
claims-guideline-analysis
regulation-analysis
procedure-analysis
evidence-comparison
```

### Tools

``` text
search_knowledge
retrieve_evidence
get_document
get_document_section
get_citations
get_product_detail
get_policy_detail
```

### Dependencies

``` text
Knowledge & GenAI Platform
Product Platform
Policy Platform
Document Repository / CMS
Knowledge Governance
```

------------------------------------------------------------------------

## A07 --- External Agent Adapter

### Purpose

Represent independently hosted external Agents through a common
lifecycle.

Examples:

``` text
Copilot Studio Agent
Dify Agent
LangGraph Agent
BU-specific Agent
Third-party Agent
```

### Runtime Interface

``` text
start_agent
send_agent_turn
resume_agent
suspend_agent
end_agent
get_agent_status
```

### Dependencies

``` text
Agent Runtime
External Agent Gateway
API Gateway
Identity / Token Exchange
Audit / Observability
```

------------------------------------------------------------------------

# 5. Skills to Create

## 5.1 Customer & Relationship Skills

``` text
S-CUS-01 customer-search
S-CUS-02 customer-brief
S-CUS-03 customer-insight
S-CUS-04 customer-timeline
S-CUS-05 relationship-summary
S-CUS-06 life-stage-analysis
S-CUS-07 customer-priority
S-CUS-08 customer-risk
S-CUS-09 customer-opportunity
```

### Main Dependencies

``` text
Customer 360
CRM
Policy 360
Interaction Platform
Decisioning
```

------------------------------------------------------------------------

# 5.2 Meeting & Sales Advisory Skills

``` text
S-SAL-01 meeting-preparation
S-SAL-02 meeting-agenda
S-SAL-03 question-generation
S-SAL-04 meeting-note-summary
S-SAL-05 meeting-action-extraction
S-SAL-06 meeting-followup
S-SAL-07 objection-preparation
S-SAL-08 solution-recommendation
```

### Main Dependencies

``` text
Customer & Sales
Customer 360
Calendar
Knowledge
Decisioning
Journey / Work
```

------------------------------------------------------------------------

# 5.3 Needs & Financial Analysis Skills

``` text
S-NEED-01 needs-analysis
S-NEED-02 coverage-gap-analysis
S-NEED-03 protection-gap-analysis
S-NEED-04 education-gap-analysis
S-NEED-05 retirement-gap-analysis
S-NEED-06 affordability-analysis
```

### Main Dependencies

``` text
Decisioning Platform
Customer Financial Data
Policy Data
Calculation Engines
```

Controlled calculations should be executed by deterministic engines, not
generated by the LLM.

------------------------------------------------------------------------

# 5.4 Product / Solution Skills

``` text
S-PROD-01 product-recommendation
S-PROD-02 product-comparison
S-PROD-03 product-explanation
S-PROD-04 benefit-explanation
S-PROD-05 eligibility-analysis
S-PROD-06 proposal-preparation
```

### Dependencies

``` text
Product Platform
Product Knowledge
Decisioning
Illustration Engine
Document Generation
Compliance
```

------------------------------------------------------------------------

# 5.5 Personalized Engagement Skills

``` text
S-ENG-01 next-best-customer
S-ENG-02 customer-interest-assessment
S-ENG-03 content-recommendation
S-ENG-04 engagement-plan
S-ENG-05 message-drafting
S-ENG-06 message-personalization
S-ENG-07 channel-recommendation
S-ENG-08 best-contact-time
S-ENG-09 followup-recommendation
S-ENG-10 campaign-personalization
```

### Dependencies

``` text
Customer 360
Interaction Data
Decisioning / NBA
Marketing Content
Consent
Communication Platform
Journey Platform
```

------------------------------------------------------------------------

# 5.6 Application & Underwriting Skills

``` text
S-UW-01 application-status
S-UW-02 application-summary
S-UW-03 application-followup
S-UW-04 missing-document-analysis
S-UW-05 underwriting-status
S-UW-06 underwriting-requirement-explanation
S-UW-07 underwriting-guideline-search
S-UW-08 application-risk-identification
```

### Dependencies

``` text
Application Platform
Underwriting System
Document Platform
UW Knowledge
Journey / Work
```

------------------------------------------------------------------------

# 5.7 Claims & Service Skills

``` text
S-CLM-01 claim-status
S-CLM-02 claim-summary
S-CLM-03 claim-requirement
S-CLM-04 claim-document-check
S-CLM-05 claim-next-step
S-CLM-06 claim-explanation

S-SVC-01 policy-service
S-SVC-02 policy-change-guidance
S-SVC-03 premium-status
S-SVC-04 renewal-support
```

### Dependencies

``` text
Claims System
Policy Admin
Claims Knowledge
Service Knowledge
Journey / Work
```

------------------------------------------------------------------------

# 5.8 Agent Performance & Coaching Skills

``` text
S-COA-01 performance-summary
S-COA-02 performance-diagnosis
S-COA-03 goal-progress-analysis
S-COA-04 activity-analysis
S-COA-05 conversion-analysis
S-COA-06 pipeline-analysis
S-COA-07 capability-assessment
S-COA-08 coaching-recommendation
S-COA-09 learning-recommendation
S-COA-10 practice-recommendation
S-COA-11 improvement-plan
```

### Dependencies

``` text
Agent 360
Sales Performance Data
Agent Growth & Coaching Platform
LMS
Knowledge
```

------------------------------------------------------------------------

# 5.9 Leader & Recruitment Skills

``` text
S-LEAD-01 team-brief
S-LEAD-02 team-performance
S-LEAD-03 agent-ranking
S-LEAD-04 agent-risk-detection
S-LEAD-05 agent-intervention-recommendation
S-LEAD-06 team-nba
S-LEAD-07 team-capacity-analysis
S-LEAD-08 leader-coaching

S-REC-01 recruitment-pipeline
S-REC-02 candidate-summary
S-REC-03 candidate-assessment
S-REC-04 candidate-ranking
S-REC-05 recruitment-followup
```

### Dependencies

``` text
Leader & Recruitment Platform
Agent 360
Decisioning
Agent Growth & Coaching
Journey / Work
```

------------------------------------------------------------------------

# 5.10 Journey & Work Skills

``` text
S-JRN-01 journey-status
S-JRN-02 journey-next-step
S-JRN-03 journey-start
S-JRN-04 journey-progress
S-JRN-05 journey-pause
S-JRN-06 journey-resume

S-WRK-01 work-item-create
S-WRK-02 work-item-summary
S-WRK-03 work-item-prioritize
S-WRK-04 work-item-complete

S-APR-01 approval-request
S-APR-02 approval-status
S-SLA-01 sla-risk
S-SLA-02 escalation-recommendation
```

### Dependencies

``` text
Journey & Work Orchestration Platform
Approval Service
SLA / Timer Service
Event Platform
```

Journey remains the authoritative owner of long-running business process
state.

------------------------------------------------------------------------

# 5.11 Knowledge & Content Skills

``` text
S-KNW-01 knowledge-search
S-KNW-02 knowledge-answer
S-KNW-03 knowledge-synthesis
S-KNW-04 regulation-search
S-KNW-05 procedure-search

S-CNT-01 content-search
S-CNT-02 content-recommendation
S-CNT-03 content-generation
S-CNT-04 content-personalization
S-CNT-05 content-compliance-check
S-CNT-06 document-generation
```

### Dependencies

``` text
Knowledge & GenAI Platform
Content CMS
Decisioning
Compliance
Document Generation
```

------------------------------------------------------------------------

# 5.12 Workbench / Productivity Skills

``` text
S-WB-01 morning-brief
S-WB-02 today-plan
S-WB-03 task-prioritization
S-WB-04 calendar-summary
S-WB-05 email-summary
S-WB-06 notification-summary
S-WB-07 daily-review
S-WB-08 weekly-review
S-WB-09 goal-planning
```

### Dependencies

``` text
Calendar
Email
Journey / Work
Decisioning / NBA
Customer & Sales
Application
Claims
Agent Goals
Notification Platform
```

------------------------------------------------------------------------

# 6. MCP / AI Tools to Create

The Tool layer should expose business-semantic capabilities rather than
raw backend endpoints.

------------------------------------------------------------------------

## 6.1 Customer / CRM Tools

``` text
T-CUS-01 search_customers
T-CUS-02 get_customer_profile
T-CUS-03 get_customer_360
T-CUS-04 get_customer_household
T-CUS-05 get_customer_relationships
T-CUS-06 get_customer_financial_profile
T-CUS-07 get_customer_interactions
T-CUS-08 get_customer_preferences
T-CUS-09 get_customer_consent
T-CUS-10 get_customer_opportunities

T-CUS-11 create_customer_note
T-CUS-12 update_customer
T-CUS-13 create_opportunity
T-CUS-14 update_opportunity
```

### Underlying Modules

``` text
Customer 360 API
CRM API
MDM / Customer Master
Interaction API
Consent API
Opportunity API
```

------------------------------------------------------------------------

# 6.2 Policy Tools

``` text
T-POL-01 get_customer_policies
T-POL-02 get_policy_detail
T-POL-03 get_policy_benefits
T-POL-04 get_policy_riders
T-POL-05 get_policy_coverage
T-POL-06 get_policy_premium
T-POL-07 get_policy_status
T-POL-08 get_policy_history
T-POL-09 get_beneficiary
```

### Underlying Modules

``` text
Policy 360 API
Policy Administration System
Policy Data Store
```

------------------------------------------------------------------------

# 6.3 Product / Illustration Tools

``` text
T-PRD-01 get_product_catalog
T-PRD-02 get_product_detail
T-PRD-03 get_product_rates
T-PRD-04 get_product_benefits
T-PRD-05 get_product_riders
T-PRD-06 calculate_premium
T-PRD-07 generate_illustration
```

### Underlying Modules

``` text
Product API
Product Master / Product Engine
Illustration Engine
Rate Engine
```

------------------------------------------------------------------------

# 6.4 Decisioning / NBA Tools

``` text
T-DEC-01 run_needs_analysis
T-DEC-02 calculate_protection_gap
T-DEC-03 calculate_education_gap
T-DEC-04 calculate_retirement_gap
T-DEC-05 calculate_affordability
T-DEC-06 check_product_eligibility
T-DEC-07 assess_customer_interest
T-DEC-08 rank_customers
T-DEC-09 score_opportunity
T-DEC-10 score_agent_risk

T-NBA-01 get_next_best_action
T-NBA-02 get_next_best_content
T-NBA-03 get_next_best_product
T-NBA-04 get_next_best_customer
T-NBA-05 get_team_nba
```

### Underlying Modules

``` text
Decisioning API
Needs Analysis Engine
Rules Engine
Scoring / ML Models
Recommendation Engine
NBA Engine
```

------------------------------------------------------------------------

# 6.5 Application / Underwriting Tools

``` text
T-APP-01 get_applications
T-APP-02 get_application_detail
T-APP-03 get_application_status
T-APP-04 get_application_requirements
T-APP-05 upload_application_document
T-APP-06 submit_application_document

T-UW-01 get_underwriting_status
T-UW-02 get_underwriting_requirements
T-UW-03 get_underwriting_decision
```

### Underlying Modules

``` text
Application API
New Business Platform
Underwriting API
Underwriting System
Document Management
```

------------------------------------------------------------------------

# 6.6 Claims Tools

``` text
T-CLM-01 get_claims
T-CLM-02 get_claim_detail
T-CLM-03 get_claim_status
T-CLM-04 get_claim_requirements
T-CLM-05 get_claim_documents
T-CLM-06 get_claim_payment
T-CLM-07 get_claim_timeline
```

### Underlying Modules

``` text
Claims API
Claims Management System
Document Management
Payment / Finance Integration
```

------------------------------------------------------------------------

# 6.7 Journey / Work / Approval Tools

``` text
T-JRN-01 get_journey
T-JRN-02 start_journey
T-JRN-03 get_journey_stage
T-JRN-04 advance_journey
T-JRN-05 pause_journey
T-JRN-06 resume_journey

T-WRK-01 get_work_items
T-WRK-02 create_work_item
T-WRK-03 update_work_item
T-WRK-04 complete_work_item

T-APR-01 get_approval
T-APR-02 create_approval
T-APR-03 approve_action
T-APR-04 reject_action

T-SLA-01 get_sla
T-SLA-02 escalate_work_item
```

### Underlying Modules

``` text
Journey Engine
Workflow Engine
Work Management
Approval Service
SLA / Timer Service
Event Bus
```

------------------------------------------------------------------------

# 6.8 Knowledge Tools

``` text
T-KNW-01 search_knowledge
T-KNW-02 retrieve_evidence
T-KNW-03 get_document
T-KNW-04 get_document_section
T-KNW-05 get_approved_content
T-KNW-06 get_citations
```

### Underlying Modules

``` text
Knowledge API
Knowledge Router
Hybrid Search
Vector Search
Keyword Search
Reranker
Document Store
Knowledge Governance
```

The Agent should not normally see separate low-level tools such as
`search_product_vector_db` and `search_compliance_vector_db`. Domain
routing should happen inside the Knowledge Platform based on policy and
Skill context.

------------------------------------------------------------------------

# 6.9 Content / Document Tools

``` text
T-CNT-01 search_content
T-CNT-02 get_content
T-CNT-03 get_content_template
T-CNT-04 generate_document
T-CNT-05 generate_proposal
T-CNT-06 generate_meeting_brief
T-CNT-07 generate_customer_summary
T-CNT-08 render_pdf
T-CNT-09 render_presentation
```

### Underlying Modules

``` text
Content CMS
Document Generation Service
Template Service
Rendering Service
Knowledge Platform
```

------------------------------------------------------------------------

# 6.10 Communication Tools

``` text
T-COM-01 draft_email
T-COM-02 send_email
T-COM-03 send_line_message
T-COM-04 send_whatsapp_message
T-COM-05 send_customer_app_message
T-COM-06 get_message_status
T-COM-07 get_delivery_status
T-COM-08 get_read_status
T-COM-09 get_click_events
T-COM-10 get_customer_reply
```

### Underlying Modules

``` text
Communication Platform
Email Gateway
LINE
WhatsApp
Customer App / AIA One
Campaign Platform
Interaction Tracking
```

Customer-facing write Tools should be classified as higher-risk and
normally require human approval.

------------------------------------------------------------------------

# 6.11 Calendar / Productivity Tools

``` text
T-CAL-01 get_calendar
T-CAL-02 get_meetings
T-CAL-03 get_availability
T-CAL-04 schedule_meeting
T-CAL-05 reschedule_meeting
T-CAL-06 cancel_meeting

T-MAIL-01 get_email
T-MAIL-02 search_email

T-TASK-01 get_tasks
T-TASK-02 create_task
T-TASK-03 update_task
T-TASK-04 complete_task
```

### Underlying Modules

``` text
Microsoft 365 / Outlook / Exchange
Calendar Platform
Enterprise Task Platform
```

------------------------------------------------------------------------

# 6.12 Agent / Coaching / Recruitment Tools

``` text
T-AGT-01 get_agent_360
T-AGT-02 get_agent_performance
T-AGT-03 get_agent_goals
T-AGT-04 get_sales_activity
T-AGT-05 get_conversion_metrics
T-AGT-06 get_pipeline_metrics
T-AGT-07 get_competency_profile
T-AGT-08 get_learning_history

T-COA-01 create_improvement_plan
T-COA-02 create_learning_assignment
T-COA-03 create_coaching_task
T-COA-04 save_roleplay_session
T-COA-05 save_roleplay_score

T-REC-01 get_candidate_360
T-REC-02 get_candidate_profile
T-REC-03 get_recruitment_pipeline
T-REC-04 create_recruitment_followup

T-TEAM-01 get_team_360
T-TEAM-02 get_team_performance
T-TEAM-03 get_agent_risk_scores
```

### Underlying Modules

``` text
Agent Growth & Coaching Platform
Leader & Recruitment Platform
Sales Performance Platform
LMS
Agent 360 / Team 360
```

------------------------------------------------------------------------

# 6.13 Agent Lifecycle Tools

``` text
T-RUN-01 start_agent
T-RUN-02 send_agent_turn
T-RUN-03 resume_agent
T-RUN-04 suspend_agent
T-RUN-05 end_agent
T-RUN-06 get_agent_status
```

### Underlying Modules

``` text
Agent Runtime
Agent Registry
Agent Session Store
External Agent Gateway
Context Policy
```

Use generic lifecycle Tools rather than creating a different
start/resume Tool for every specialized Agent.

------------------------------------------------------------------------

# 7. Knowledge Domains Required

Recommended logical Knowledge Domains:

``` text
K01 PRODUCT
K02 SALES_ADVISORY
K03 UNDERWRITING
K04 CLAIMS_SERVICE
K05 COMPLIANCE_REGULATORY
K06 OPERATIONS_SOP
K07 COACHING_LEARNING
K08 RECRUITMENT_LEADERSHIP
K09 MARKETING_APPROVED_CONTENT
K10 CORPORATE
K11 MARKET_EXTERNAL
K12 PERSONAL_TEAM
```

These are logical domains. They do not require 12 physically independent
vector databases.

A smaller number of physical stores/indexes can support them through
metadata, authorization and Knowledge Router policies.

Operational entities such as:

``` text
Customer
Policy
Claim
Application
Interaction
Journey
Task
Agent Performance
Candidate
```

should remain structured operational/semantic data rather than being
treated as RAG Knowledge Bases.

------------------------------------------------------------------------

# 8. Major Business Applications / Platforms Required

## B01 --- AI Workbench & Experience

Provides:

``` text
Agent Workbench
Leader Workbench
Goal Composer
AI Plan
Execution Progress
Dynamic Workspace
Human + AI Task Queue
Approval UI
Explainability
Interrupt / Resume UX
```

------------------------------------------------------------------------

## B02 --- Customer & Sales Platform

Provides:

``` text
Customer 360
Customer Relationship
Interactions
Opportunities
Meeting
Sales Activity
Engagement History
Customer Timeline
```

Key APIs:

``` text
Customer 360 API
Interaction API
Opportunity API
Meeting API
```

------------------------------------------------------------------------

## B03 --- Agent Growth & Coaching Platform

Provides:

``` text
Agent Goals
Performance
Competency
Coaching
Role Play Results
Improvement Plan
Learning Recommendation
```

------------------------------------------------------------------------

## B04 --- Leader & Recruitment Platform

Provides:

``` text
Team 360
Team Performance
Candidate 360
Recruitment Pipeline
Leader Coaching
Recruitment Activities
```

------------------------------------------------------------------------

## B05 --- Journey & Work Orchestration Platform

Provides authoritative:

``` text
Journey State
Stage / Transition
Work Item
Timer / SLA
Approval
Escalation
Retry
Long-running Process State
```

------------------------------------------------------------------------

## B06 --- Decisioning & NBA Platform

Provides controlled business decisions:

``` text
Needs Analysis
Protection Gap
Affordability
Customer Ranking
Interest Assessment
Opportunity Scoring
Product Recommendation
Content Recommendation
Next Best Action
Next Best Customer
Agent Risk
```

------------------------------------------------------------------------

## B07 --- Knowledge & Generative AI Platform

Provides:

``` text
Knowledge Ingestion
Document Processing
Knowledge Router
Hybrid Retrieval
Reranking
Citation
Knowledge Governance
Content Generation
Document Generation
```

------------------------------------------------------------------------

## B08 --- 360 Data & Semantic Platform

Provides governed structured views:

``` text
Customer 360
Agent 360
Candidate 360
Team 360
Policy View
Claim View
Application View
Interaction View
Features
Semantic Definitions
Identity Resolution
Consent
Lineage
```

------------------------------------------------------------------------

## B09 --- Engagement / Communication Platform

Provides:

``` text
Content Delivery
LINE
WhatsApp
Email
Customer App Messaging
Delivery Tracking
Open / Click / Reply Events
Consent Enforcement
```

------------------------------------------------------------------------

## B10 --- Product / Illustration Platform

Provides:

``` text
Product Catalog
Product Master
Benefits
Riders
Rates
Eligibility
Premium Calculation
Illustration
```

------------------------------------------------------------------------

## B11 --- Application / Underwriting Platform

Provides:

``` text
Application
Requirements
Documents
UW Status
UW Requirements
UW Decision
```

------------------------------------------------------------------------

## B12 --- Claims / Service Platform

Provides:

``` text
Claims
Claim Status
Claim Requirements
Claim Documents
Claim Payment
Policy Service
```

------------------------------------------------------------------------

## B13 --- Identity / AI Control Platform

Provides:

``` text
SSO
Authentication
Authorization
Fine-grained Permissions
Consent Enforcement
Risk Policy
Guardrails
Audit
Model Gateway
AI Observability
Tool Authorization
```

------------------------------------------------------------------------

# 9. Enterprise Systems / External Dependencies

The Business Applications above may integrate with existing enterprise
systems rather than replace them.

Typical dependencies:

``` text
CRM
Customer Master / MDM
Policy Administration System
Product Master
Illustration Engine
New Business / Application System
Underwriting System
Claims System
Document Management
Content Management System
LMS
Data Warehouse / Lakehouse
Identity / SSO
Microsoft 365 / Outlook / Exchange
LINE
WhatsApp
Customer Mobile App / AIA One
Notification Platform
Payment / Finance Systems
Enterprise Event Bus
API Gateway
```

------------------------------------------------------------------------

# 10. Agent → Skill → Tool → System Examples

## Example 1 --- Sales Advisory

``` text
Sales Advisory Agent
        ↓
needs-analysis Skill
        ↓
run_needs_analysis Tool
        ↓
Decisioning API
        ↓
Needs Analysis Engine
```

------------------------------------------------------------------------

## Example 2 --- Customer Brief

``` text
Sales Advisory Agent
        ↓
customer-brief Skill
        ↓
get_customer_360
get_customer_policies
get_customer_interactions
        ↓
Customer 360 API
Policy API
Interaction API
        ↓
CRM
Policy Admin
Interaction Data
```

------------------------------------------------------------------------

## Example 3 --- Personalized Engagement

``` text
Main Agent
        ↓
engagement-plan Skill
        ↓
customer-interest-assessment
content-recommendation
message-personalization
compliance-check
        ↓
Decision / Knowledge / Content Tools
        ↓
Decisioning
Knowledge Platform
Content CMS
        ↓
Human Approval
        ↓
send_customer_message
        ↓
Communication Platform
        ↓
LINE / WhatsApp / Email / Customer App
```

------------------------------------------------------------------------

## Example 4 --- Coaching

``` text
Deep Coaching Agent
        ↓
performance-diagnosis Skill
        ↓
get_agent_performance
get_sales_activity
get_conversion_metrics
        ↓
Agent Performance APIs
        ↓
Sales / Performance Data
```

Then:

``` text
Deep Coaching Agent
        ↓
improvement-plan Skill
        ↓
create_improvement_plan
create_learning_assignment
        ↓
Coaching Platform
LMS
```

------------------------------------------------------------------------

## Example 5 --- Claim Interrupt

``` text
Sales Advisory Agent active
        │
        │ User asks unrelated claim question
        ▼
Main Orchestrator
        ↓
Suspend Sales Advisory Agent
        ↓
claim-status Skill
        ↓
get_claim_status Tool
        ↓
Claims API
        ↓
Claims System
        ↓
Answer
        ↓
Resume Sales Advisory
```

------------------------------------------------------------------------

# 11. Dependency Matrix by Agent

  -----------------------------------------------------------------------
  Agent                   Primary Skills          Primary Business
                                                  Application
                                                  Dependencies
  ----------------------- ----------------------- -----------------------
  Main Agent              Cross-domain            Orchestration,
                          orchestration           Identity, Permission,
                                                  Journey, Decisioning,
                                                  Knowledge

  Sales Advisory Agent    Customer, Needs,        Customer & Sales,
                          Product, Meeting,       Policy/Product,
                          Proposal                Decisioning, Knowledge,
                                                  Journey

  Role Play Agent         Scenario, Simulation,   Coaching, Knowledge,
                          Evaluation, Coaching    Product, LMS

  Deep Coaching Agent     Performance, Diagnosis, Agent Growth, Agent
                          Improvement             360, LMS, Sales
                                                  Performance

  Recruitment / Leader    Candidate, Team,        Leader/Recruitment,
  Agent                   Recruitment, Coaching   Agent 360, Decisioning

  Knowledge Expert Agent  Research, Synthesis,    Knowledge, Product,
                          Explanation             Policy, Document/CMS

  External Agent Adapter  External capability     External Agent Gateway,
                          dependent               Identity, Audit
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 12. Tool Risk Classification

Every Tool should carry an execution risk class.

## READ_ONLY

Examples:

``` text
get_customer_360
get_policy_detail
get_claim_status
search_knowledge
```

Normally auto-executable after authorization.

## LOW_RISK_WRITE

Examples:

``` text
create_customer_note
create_internal_task
create_followup
```

Can normally be automated subject to policy.

## CUSTOMER_FACING

Examples:

``` text
send_customer_message
send_email
schedule_customer_meeting
```

Normally requires explicit human approval.

## REGULATED_ACTION

Examples:

``` text
submit_application
submit_proposal
accept_customer_instruction
execute_policy_change
```

Requires strong deterministic controls and explicit approval.

------------------------------------------------------------------------

# 13. Recommended MVP Build Scope

A practical first release should focus on a small number of end-to-end
flows rather than implementing the full target inventory.

## Agents

``` text
Main Agent / Orchestrator
Role Play Agent

Optional:
Sales Advisory Agent
```

If Sales Advisory multi-turn functionality is central to the initial
release, it should be included early.

## Core Skills

``` text
morning-brief
today-plan

customer-brief
customer-insight

meeting-preparation
meeting-note-summary
meeting-followup

needs-analysis
coverage-gap-analysis
affordability-analysis

product-recommendation
product-comparison
proposal-preparation

content-recommendation
message-drafting
engagement-plan

application-followup
claim-status

performance-summary
performance-diagnosis

knowledge-answer
compliance-check
```

## Core MCP Tools

``` text
get_customer_360
get_customer_interactions
get_customer_consent

get_customer_policies
get_policy_detail

get_product_catalog
get_product_detail
calculate_premium

run_needs_analysis
calculate_protection_gap
calculate_affordability
get_next_best_action
get_next_best_product
get_next_best_content

get_application_status
get_underwriting_status
get_claim_status

search_knowledge
retrieve_evidence
get_approved_content

get_work_items
create_work_item
create_approval

get_calendar
schedule_meeting

generate_proposal
send_customer_message

get_agent_performance
get_agent_goals

start_agent
send_agent_turn
resume_agent
suspend_agent
end_agent
```

## MVP Knowledge Domains

``` text
PRODUCT
SALES_ADVISORY
COMPLIANCE_REGULATORY
UNDERWRITING
CLAIMS_SERVICE
OPERATIONS_SOP
COACHING_LEARNING
MARKETING_APPROVED_CONTENT
```

## MVP Business Application Dependencies

``` text
Customer & Sales
Journey & Work
Decisioning & NBA
Knowledge & GenAI
360 Data & Semantic
Product / Policy
Application / Underwriting
Claims
Engagement / Communication
Identity / AI Control
```

------------------------------------------------------------------------

# 14. Recommended Ownership Model

The implementation should be divided by stable capability ownership
rather than by individual Agents.

``` text
Agentic Platform Team
├─ Main Agent Runtime
├─ Agent Runtime
├─ Skill Runtime
├─ Tool / MCP Gateway
├─ Context Policy
├─ Agent / Skill / Tool Registry
└─ Approval / Runtime Governance

Customer & Sales Team
├─ Customer Skills
├─ Meeting Skills
├─ Engagement Skills
└─ Customer APIs

Decisioning Team
├─ Needs Analysis
├─ Scoring
├─ Recommendation
├─ NBA
└─ Decision APIs

Knowledge / GenAI Team
├─ Knowledge Platform
├─ Knowledge Router
├─ Retrieval / Rerank
├─ Content Generation
└─ Knowledge Tools

Journey Team
├─ Journey Engine
├─ Work Management
├─ Approval
├─ SLA
└─ Journey Tools

Coaching / Leader Team
├─ Coaching Skills
├─ Role Play
├─ Recruitment Skills
└─ Leader Skills

Integration Team
├─ CRM
├─ Policy
├─ UW
├─ Claims
├─ Calendar / Email
└─ Messaging Channels
```

This avoids duplicating the same integration and business logic inside
multiple Agents.

------------------------------------------------------------------------

# 15. Final Design Rule

The target architecture should consistently follow:

``` text
Agent
  │
  │ decides what capability is needed
  ▼
Skill
  │
  │ implements a bounded business capability
  ▼
MCP / AI Tool
  │
  │ provides governed deterministic access
  ▼
Business Application API
  │
  │ owns authoritative business behavior
  ▼
Enterprise System / Data / Engine
```

For complex business work:

``` text
Agent → Skill → Multiple Tools → Multiple Business Applications
```

For simple read-only requests:

``` text
Agent → Read Tool → Business Application
```

For independent multi-turn specialist work:

``` text
Main Agent → Specialized Agent → Skills → Tools
```

For customer-facing or regulated actions:

``` text
Agent
  ↓
Skill / Tool Request
  ↓
Deterministic Runtime Policy
  ↓
Human Approval
  ↓
Tool Execution
```

The architecture should therefore optimize for **reusable Skills and
governed Tools**, rather than maximizing the number of autonomous
Agents.
