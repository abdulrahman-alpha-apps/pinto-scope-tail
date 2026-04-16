---
title: "backend"
publish: true
---

# Pinto Backend

## What the Backend Is

The Pinto backend is the operational middle layer that sits between the Pinto Agent, the user-facing Pinto App, the internal Accountant Dashboard, and the connected accounting systems: Wafeq and Zoho Books.

It is not just a thin API layer. It is the system that gives Pinto reliability, memory, control, and safety. The backend translates conversational actions into structured accounting operations, stores the operational state needed across channels, synchronizes external systems, and enforces the rules that keep the Agent from acting unsafely or inconsistently.

In practical terms, the backend does four jobs:

1. It integrates with Wafeq and Zoho Books
2. It supports the Agent with structured context, tools, and guardrails
3. It stores Pinto-owned operational data and workflow state
4. It keeps all surfaces in sync: WhatsApp Agent, Pinto App, Accountant Dashboard, and accounting system

---

## Core Role in the Pinto Architecture

Pinto has three visible product surfaces:

- The Agent on WhatsApp
- The Pinto App for business users
- The Accountant Dashboard for the internal operations team

The backend is the shared control plane behind all three.

When a user sends a message to the Agent, the backend helps interpret the request into a valid financial workflow, fetches or stores the required data, calls the right accounting integration, and records the resulting state. When the Agent cannot safely complete a flow, the backend creates and manages the fallback path through tasks, inquiries, reminders, and handoff to accountants.

This makes the backend the source of operational coordination across the product, rather than leaving each surface to manage its own state independently.

---

## Main Responsibilities

### 1. Accounting System Integration Layer

The backend is responsible for all structured communication with Wafeq and Zoho Books.

This includes:

- Managing authenticated connections to the user's accounting system
- Abstracting provider-specific API differences behind a Pinto-friendly internal model
- Creating and updating records such as contacts, bills, invoices, payments, and reports
- Reading accounting data back so the Agent and App can answer questions and show live state
- Handling sync jobs, retries, failures, and reconciliation when external APIs are unavailable or inconsistent

Wafeq and Zoho are not treated as passive destinations. They are active systems of record for accounting data. The backend therefore has to maintain a clean contract between Pinto workflows and each provider's API model.

That means the backend should normalize concepts like:

- Contacts: customer, vendor, supplier
- Sales documents: invoice, credit note, payment against invoice
- Purchase documents: bill, expense, payment against bill
- Taxes and VAT treatment
- Currencies, references, due dates, and payment status

This normalization is what allows the Agent to behave consistently even when the underlying provider is different.

---

### 2. Agent Support Layer

The Agent is the conversational interface, but the backend provides the structure that makes the Agent usable in production.

The backend supports the Agent by:

- Providing customer, vendor, invoice, expense, and task context
- Exposing safe internal actions for the Agent to call
- Validating whether the Agent has enough information to proceed
- Persisting conversation-linked workflow state across messages and sessions
- Recording what the Agent attempted, what succeeded, and what failed
- Deciding when to continue automatically, ask a follow-up question, or escalate

The Agent should not directly improvise accounting actions against Wafeq or Zoho. The backend should mediate those actions through controlled operations with validation rules and explicit status handling.

This is one of the backend's most important jobs: converting flexible natural-language intent into bounded, auditable system behavior.

---

### 3. Pinto-Owned Data Storage

Not all important data should live only inside Wafeq or Zoho. Pinto needs its own operational database because it must track workflow state that external accounting systems do not understand.

The backend should store and manage data such as:

- Users, organizations, and connected accounting provider metadata
- WhatsApp identity mapping and app authentication state
- Conversation threads and workflow progress
- OCR extraction output and normalized document payloads
- Tasks created by the Agent or by internal teams
- Inquiry records and user responses
- Reminder state and follow-up scheduling
- Audit logs of actions taken by the Agent, backend, and accountants
- Integration sync status, last successful fetch, retry state, and failure reasons
- Cached or denormalized operational views needed by the app and dashboard

This Pinto-owned layer is what lets the product work as a coordinated system rather than as a collection of loose API calls into third-party tools.

---

### 4. Workflow Orchestration and State Management

The backend is responsible for managing end-to-end operational flows across asynchronous systems.

Examples:

- A user uploads a receipt on WhatsApp
- OCR extracts the data
- The Agent asks one clarification question
- The backend stores the incomplete draft state
- The user replies later
- The backend resumes the flow with the existing context
- A bill is created in Wafeq or Zoho
- If something fails, a task is created for the accountant team

This orchestration layer is necessary because Pinto flows are rarely single-request, single-response transactions. They are often multi-step workflows with pauses, retries, handoffs, and external dependencies.

The backend therefore owns status progression for objects such as:

- Draft
- Awaiting clarification
- Ready to sync
- Synced
- Failed
- Escalated
- Waiting on user
- Resolved

These statuses give Pinto a durable operational backbone and make behavior consistent across Agent, App, and Dashboard.

---

### 5. Guardrails and Operational Safety

The backend is also the guardrail layer for the Agent.

Its job is not only to help the Agent do things, but to stop the Agent from doing the wrong thing. In Pinto, that is essential because the product operates on financial data and may create accounting records with compliance implications.

Backend guardrails should include:

- Required-field validation before any accounting action is executed
- Provider-aware validation for Wafeq and Zoho payloads
- VAT and tax rule enforcement where applicable
- Duplicate detection for bills, invoices, payments, and uploads
- Confidence thresholds for OCR or entity matching before auto-posting
- Limits on repeated retries and follow-up loops
- Clear escalation rules when confidence is low or data is conflicting
- Idempotent write behavior so the same user action does not create duplicate accounting entries
- Auditability for every important decision and mutation

The backend should be the place where Pinto's business rules live, not the Agent prompt alone.

---

## Communication Paths

### Agent ↔ Backend

The Agent communicates with the backend to:

- fetch user and organization context
- retrieve relevant accounting data
- store extracted and confirmed workflow data
- execute guarded actions
- create tasks or inquiries when automation cannot continue

The backend returns structured outcomes, not just raw data. For example:

- success with created record ID
- needs clarification with missing fields
- blocked due to mismatch or ambiguity
- external integration failure
- escalated to accountant team

This lets the Agent respond naturally while still behaving deterministically underneath.

### Backend ↔ Wafeq / Zoho

The backend communicates with Wafeq and Zoho to:

- connect and refresh authentication
- read master data and transactional data
- create and update accounting records
- reconcile status after writes
- pull reports and balances when needed

Because the provider APIs differ, the backend should expose one internal domain model and keep provider-specific mapping logic inside dedicated integration modules.

### Backend ↔ Pinto App

The Pinto App depends on the backend for:

- WhatsApp-based identity and OTP-linked access flows
- task and inquiry retrieval
- open expense and invoice state
- dashboard summaries and recent activity
- status updates when users respond or resolve issues

The App should read from backend-managed operational views rather than having to understand Wafeq or Zoho directly.

### Backend ↔ Accountant Dashboard

The Accountant Dashboard depends on the backend for:

- task inbox and filtering
- task detail payloads with pre-filled extracted data
- inquiry creation and response tracking
- status transitions such as `open`, `in_progress`, `waiting_on_user`, and `done`
- final posting or replay of corrected data into the accounting system

This keeps accountants working inside a Pinto workflow rather than manually stitching together context from external systems.

---

## Database Sync Responsibility

One of the backend's most important responsibilities is database synchronization.

Pinto needs to stay aligned with two realities at the same time:

- the external accounting system, which holds the accounting ledger
- Pinto's own operational database, which holds workflow state and product context

The backend is responsible for keeping those two layers aligned.

That includes:

- pulling fresh accounting data needed by the Agent and App
- updating Pinto-managed views after successful writes
- detecting stale or conflicting records
- retrying failed sync jobs
- reconciling partial failures
- ensuring the UI does not show states that contradict the accounting system

In other words, the backend should not only write to Wafeq or Zoho. It should also maintain a dependable synchronized view of the user's financial world so the Agent and product surfaces can operate quickly and consistently.

---

## What the Backend Owns vs What It Does Not Own

The backend owns:

- workflow state
- task lifecycle
- inquiry lifecycle
- sync status
- authentication and integration state
- Pinto-specific operational data
- business rules and guardrails
- audit history

The backend does not replace:

- Wafeq or Zoho as the accounting system of record
- the Agent as the conversational interface
- the App as the user-facing resolution and visibility layer
- the Accountant Dashboard as the internal human operations layer

Its role is to coordinate and enforce, not to become the UI and not to bypass the accounting platforms.

---

## Why This Layer Matters

Without this backend layer, Pinto would be fragile:

- The Agent would make direct, unsafe, provider-specific decisions
- State would be lost across conversations and handoffs
- Tasks and inquiries would not have a reliable lifecycle
- The App and Dashboard would drift out of sync
- Wafeq and Zoho failures would leak directly into the user experience

With a strong backend, Pinto becomes reliable and scalable. The Agent can stay conversational, the accountants can work with structured context, and the product can maintain consistent financial state across every surface.

That is the core purpose of the Pinto backend: it is the trusted middle layer that turns AI-driven accounting workflows into a controlled, synchronized, and auditable system.
