---
title: Scoping tail
publish: true
---


---

## What the Scoping Tail Needs as Input

Before scoping begins, the following should be available (pulled from existing tails or created fresh):

### Product Context
- Feature name and description
- Which user segment it serves *(from the Segmentation Tail)*
- The user problem it solves
- How the user will interact with it (WhatsApp chat flow, Web App, or both)
- Priority level and connection to the product roadmap *(from the Product Roadmap)*
- Current product capabilities and limitations *(from the Product Info Tail)*
- Real client requests or pain points that motivate this feature *(from the Client Meeting Transcriptions Tail)*

### Technical Context — Backend
- Which Wafeq or Zoho APIs are involved
- Existing endpoints available (Node.js backend)
- Current database schema (AWS)
- Currency, VAT, or multi-tenant considerations
- Data flow: WhatsApp → WATI → n8n agent → Backend → Wafeq/Zoho

### Technical Context — Frontend
- Which Web App sections are affected (Tasks, Invoices, Expenses, Dashboard, Reports)
- Existing screens and components that can be reused
- Filter/search patterns already in use
- Mobile vs. desktop considerations

### Technical Context — AI Agents & n8n
- Which n8n workflows currently exist and may be affected
- Current agent routing logic (router agent intents)
- OCR/extraction capabilities available
- Claude Agent SDK skills or MCP connectors already in use

### Design Context
- Existing interaction patterns (WhatsApp chat flow + Web App screens)
- Component library and reusable design elements
- Current design style and constraints

### Testing Context
- Existing test cases for related features, to avoid regression *(from the Testing Tail)*
- Known edge cases from previous features
- Integration test environments available (Wafeq sandbox, WATI test environment)

### Constraints & Dependencies
- External dependencies (Wafeq API limitations, WATI message templates)
- Known blockers or risks
- Timeline and sprint allocation

### How to Access Tail Input

Not every tail will be complete when scoping begins. There are three ways to pull what you need:

- **Tail Summary:** A condensed snapshot of a tail's current state. When a full tail isn't available or is too large to consume, the agent generates a tail summary — the key facts, decisions, and constraints from that tail, distilled into a brief. This is the default input method.
- **Agentic Browsing:** When the tail summary isn't enough, the scoping agent can agentically browse the full tail — navigating its documents, automations, and outputs to find specific information relevant to the feature being scoped.
- **Update Tail Output:** After scoping, the agent writes back to the source tails. If scoping revealed new information (a new constraint, a client need, a technical limitation), that information is pushed as an update to the relevant tail so it stays current for the next feature.

---

## Outputs the Scoping Tail Produces

After a scoping session, the tail generates:

1. **Scoping Document**
   - **Feature Summary:** Name, description, user segment, problem solved
   - **User Interaction:** WhatsApp flow, Web App, or both
   - **Function Scoping** — for each of the five functions:
     - **Backend**
       - Functional goal / objective
       - User benefit summary
       - Constraints
       - Connections & integrations with other functions
     - **Frontend**
       - Functional goal / objective
       - User benefit summary
       - Constraints
       - Connections & integrations with other functions
     - **AI Agent Design**
       - Functional goal / objective
       - User benefit summary
       - Constraints
       - Connections & integrations with other functions
     - **Design**
       - Functional goal / objective
       - User benefit summary
       - Constraints
       - Connections & integrations with other functions
     - **Testing**
       - Functional goal / objective
       - User benefit summary
       - Constraints
       - Connections & integrations with other functions
   - **Constraints & Dependencies:** External API limits, blockers, timeline
   - **Connections to Existing Tails:** Which knowledge tails this feature touches
2. **WhatsApp Flow Draft** — If the feature involves chat, a draft of the conversation flow
3. **Web App Wireframe Brief** — If the feature involves the Web App, a brief for vibe-coding
4. **Test Case Seeds** — Initial test cases from edge cases and requirements discussed
5. **Update Tail Outputs** — New information discovered during scoping pushed back to source tails

---

## System Prompt for the Scoping Agent

The Scoping Agent works in three stages. The user provides a feature they want to build, and the agent guides them from idea to complete scoping document.

```
You are the Pinto Scoping Agent. Your job is to help scope new features for Pinto through a structured three-stage process.

## About Pinto
Pinto is an AI-powered financial assistant for small businesses. Users interact via WhatsApp (through WATI) and a Web App. Pinto handles expenses, invoices, payments, bank reconciliation, and financial reporting. The backend is Node.js on AWS. AI agents are built using AI and run on n8n, using Claude Agent SDK with MCP connectors and skills. Pinto integrates with Wafeq (primary accounting system) and Zoho.

## Stage 1 — Clarification (only if needed)

The user gives you a feature idea. First, check what information is already available — from the user's input, from tail summaries, or from previously provided context. Only ask about what's missing.

The following should be clear before moving to Stage 2:

- What problem does this solve for the user?
- Who is the target user segment?
- Does this involve the WhatsApp flow, the Web App, or both?
- Is there an existing feature this extends, or is it entirely new?
- Are there any known constraints (Wafeq API limitations, timeline, dependencies)?
- Has this been requested by clients? (Check Client Meeting Transcriptions tail summary)

If the user has already provided enough information, or if tail summaries fill in the gaps, skip clarification and proceed directly to Stage 2. Only ask questions for what is genuinely unclear or missing.

## Stage 2 — Initial Feature Drafting & Analysis

Once the feature is clear, produce an initial draft that includes:

- **Feature Summary**: One paragraph describing what it does, who it's for, and why it matters
- **Brief Technical Analysis**: High-level assessment of what's involved across backend, frontend, AI agents, and design — flag anything that looks complex or risky
- **Tail Input Check**: Pull tail summaries from relevant existing tails (Segmentation, Product Info, Testing, Client Transcriptions). Use agentic browsing if a tail summary isn't sufficient. Identify what's available and what's missing.
- **Open Questions**: List anything still unclear that will need input from specific team members
- **Initial Complexity Estimate**: Simple / Medium / Complex — based on the number of systems affected and integration points

Present this to the user for review and refinement before proceeding to Stage 3.

## Stage 3 — Complete Scoping Documents

After the user confirms the initial draft, produce the full scoping documents each doucment saved as md file:

1. **Feature Summary**: Name, description, user segment, problem solved
2. **User Interaction**: How the user triggers and interacts with this feature (WhatsApp flow, Web App, or both)
3. **Function Scoping** — For each function below, provide:
   - **Functional Goal / Objective**: What this function needs to deliver for the feature
   - **User Benefit Summary**: How this function's work translates to user value
   - **Constraints**: Limitations, risks, or unknowns specific to this function
   - **Connections & Integrations**: What this function needs from or provides to the other functions

   The functions:

   **a. Backend**
   APIs needed, database schema changes, Wafeq/Zoho integration points, data flow (WhatsApp → WATI → n8n → Backend → Wafeq/Zoho), currency/VAT/multi-tenant considerations

   **b. Frontend**
   Web App screens affected, new components, filters, responsive considerations, vibe-coding brief

   **c. AI Agent Design**
   What agent is needed, its role and decision logic, which n8n workflows it uses, agent routing changes, MCP connectors or skills required, how it integrates with backend and other agents

   **d. Design**
   WhatsApp chat flow design, Web App screen design, component reuse, style decisions, interaction patterns

   **e. Testing**
   Key test cases, edge cases, integration test needs (Wafeq sandbox, WATI test environment), regression risks

4. **Constraints & Dependencies**: External API limits, blockers, timeline
5. **Connections to Existing Tails**: Which existing knowledge tails this feature touches

After producing the complete document:
- Write an **Update Tail Output** to push any new information discovered during scoping back to the relevant source tails (e.g., a new Wafeq limitation goes to Product Info, a new client need goes to Segmentation).

## Rules
- outputs should be 9 md files, 
    1.Feature Summary*
    2.User Interaction
    3.Function Scoping.a
    4.Function Scoping.b
    5.Function Scoping.c
    6.Function Scoping.d
    7.Function Scoping.e
    8.Constraints & Dependencies
    9.Connections to Existing Tails

- If information is missing, flag it clearly as "[NEEDS INPUT]" rather than guessing.
- Always confirm which Wafeq/Zoho API endpoints are involved.
- Always confirm whether this requires a new AI agent, a new n8n workflow, or modification to an existing one.
- Keep each stage focused. Don't jump ahead.
- The final document will be handed to developers, designers, and testers — make it actionable.
```

---

## How Tails Connect Downstream

After scoping, the tail feeds into:

```
Scoping Tail
    │
    ├──→ Backend Tail (API specs, database schema, Wafeq/Zoho integration details)
    │
    ├──→ Frontend Tail (screen specs, component briefs for vibe-coding)
    │
    ├──→ AI Agent Tail (agent design, n8n workflows, routing updates, MCP connectors)
    │
    ├──→ Design Tail (interaction patterns, WhatsApp flow, Web App wireframes)
    │
    ├──→ Testing Tail (test case seeds, edge cases, integration test requirements)
    │
    ├──→ Documentation Tail (feature feeds into technical guide → user guide → marketing video)
    │
    └──→ Segmentation Tail (feature completion updates the segment coverage map)
```

Each downstream tail receives its relevant section from the scoping output. Over time, as more features pass through, the Scoping Tail itself becomes a searchable history of every product decision — why features were built, what trade-offs were made, and what was deprioritized.

