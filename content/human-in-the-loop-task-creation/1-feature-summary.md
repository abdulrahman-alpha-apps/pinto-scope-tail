---
title: "1. Feature Summary"
publish: true
---

# Feature Summary — Human-in-the-Loop Task Creation

## Feature Name
Human-in-the-Loop (HITL) Task Creation

## One-Line Description
A standardized, agent-wide escalation mechanism that creates an accountant task whenever an AI agent fails to complete a workflow or a user explicitly disputes the agent's output.

---

## What It Does

When any of Pinto's AI agents — Router, Expense, Invoice, Payment, Reporting, or Onboarding — reaches a point it cannot resolve automatically, it calls the backend API (`POST /ai/organizations/tasks`) to create a structured task for the accountant team. The same mechanism fires when a user actively objects to something the agent has done (for example, wanting to change a VAT treatment the agent applied automatically).

The feature does not build new infrastructure. The backend endpoint, the accountant task inbox, and the inquiry system already exist. This feature defines the complete trigger matrix — every condition across every agent that should produce a task — and standardizes the n8n implementation pattern so that all agents behave consistently when escalation is required.

---

## Problem It Solves

Currently, task creation is inconsistently implemented across agents:

- The **Expense** and **Payment** agents have dedicated fallback tools (`Fallback dummy`, `Payment Fallback`) that already call the task API.
- The **Invoice** agent has an `exit` tool that closes a conversation cleanly but does not create an accountant task when exiting due to failure.
- The **Reporting**, **Onboarding**, and **Router** agents have no documented task-creation path at all.
- **User objections** — where a user disagrees with an agent's decision and wants a different outcome — have no defined handling path and no corresponding task category.

Without a complete escalation model, certain failure modes are silent: the user gets no resolution, the accountant team is never notified, and the financial record is left incomplete.

---

## User Segments

| Segment | Relevance |
|---|---|
| Small business owners (primary WhatsApp users) | Experience the agent failure or raise the objection; receive the WhatsApp escalation notification |
| Pinto accountant team | Receives the task in the Accountant Dashboard; resolves and posts the corrected entry |

---

## Surfaces Involved

| Surface | Role |
|---|---|
| WhatsApp (via WATI) | User is notified when escalation occurs; receives resolution confirmation when accountant completes the task |
| n8n AI workflows | Each agent triggers task creation at the correct point in its workflow |
| Backend API | `POST /ai/organizations/tasks` — the single endpoint all agents call |
| Accountant Dashboard (Retool) | Receives all created tasks; accountant works through them and resolves |
| Pinto App (Web) | User can see open tasks and respond to accountant inquiries if needed |

---

## Task Categories Defined by This Feature

| Category Value | Meaning |
|---|---|
| `issue` | Agent failed due to bad, missing, or ambiguous data — accountant must review and correct |
| `normal_task` | Agent could not complete a routine action cleanly — accountant completes the step |
| `error` | A system-level failure (API error, timeout, processing crash) — technical or accountant review |
| `user_objection` | *(New)* User explicitly disputes the agent's output and requests a different outcome |

The `user_objection` category is new and must be added to the backend `CreateTaskDto` schema enum.

---

## Entity Types Covered

| Entity Value | Created By |
|---|---|
| `bill_ocr` | Expense agent (OCR-driven receipt processing) |
| `bill_manual_entry` | Expense agent (manual expense input) |
| `invoice` | Invoice agent |
| `payment` | Payment agent |
| `report` | Reporting agent |
| `general_task` | Router agent, Onboarding agent, cross-agent failures |

---

## What Happens After a Task Is Created

1. The backend creates the task record and returns the task ID.
2. The agent updates the thread status to `escalated` in Postgres/Data Tables.
3. The user receives a WhatsApp message: *"Your request is being reviewed by our team. We'll get back to you shortly."*
4. The accountant sees the task appear in their inbox (Retool) with all available pre-filled data.
5. The accountant resolves the task — posting the corrected entry to Wafeq or Zoho — and the user receives a WhatsApp confirmation: *"We've completed your request. No action needed."*
6. If the accountant needs more information from the user, they send an Inquiry. The task moves to `waiting_on_user` until the user responds via the Pinto App.

---

## Trigger Model Summary

| Agent | Trigger Type | Examples |
|---|---|---|
| Router | System error, unrecognized intent after retries | Webhook crash, intent loop |
| Expense | OCR failure, vendor mismatch, missing fields, user objection | 3 failed OCR attempts, TRN mismatch, user disputes VAT treatment |
| Invoice | Customer not matched, item not resolved, API failure, user objection | Customer create fails, invoice POST fails, user disputes line items |
| Payment | Contact not matched, account not found, bulk allocation failure, user objection | Bank statement parse fails, payment POST fails |
| Reporting | Contact/vendor not matched after retries, user requests unsupported report type | Statement of account vendor not resolved |
| Onboarding | OAuth failure, accounting system misconfiguration, tax setting error | Wafeq connect fails, user set wrong VAT values in accounting system |

Full detail — including request body for each case — is in `2. All Create Task Cases.md`.

---

## Scope Boundaries

**In scope:**
- Defining all HITL trigger conditions per agent
- Standardizing the `POST /ai/organizations/tasks` request body per case
- Adding `user_objection` as a new task category
- Standardizing the n8n task-creation node pattern across all agents
- Defining the WhatsApp message sent to the user at escalation

**Out of scope:**
- Changes to the Accountant Dashboard UI
- Changes to the Inquiry system
- Changes to how accountants resolve tasks
- Changes to Wafeq or Zoho integration (the task payload carries the extracted data; posting happens after resolution)

---

## Dependencies

| Dependency | Status |
|---|---|
| `POST /ai/organizations/tasks` backend endpoint | Already live |
| `CreateTaskDto` schema — add `user_objection` to category enum | Backend change required |
| Expense agent Fallback dummy | Already implemented |
| Payment agent Payment Fallback | Already implemented |
| Invoice agent — HITL fallback tool | Needs to be built |
| Reporting agent — HITL fallback tool | Needs to be built |
| Onboarding agent — HITL fallback tool | Needs to be built |
| Router agent — wire `creating_a_task1` node to defined conditions | Needs verification and wiring |
