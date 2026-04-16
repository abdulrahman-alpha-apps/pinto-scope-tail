---
title: "04-pinto-accountant-dashboard"
publish: true
---

# The Accountant Dashboard

## What is the Accountant Dashboard?

The Accountant Dashboard is Pinto's internal back-office tool, built on Retool, used exclusively by Pinto's accountant team. It is the human-in-the-loop layer of the platform — the place where expert accountants step in to review, correct, and complete financial transactions that the AI Agent could not resolve automatically.

The dashboard is not visible to business users. It is an operational tool designed for speed and precision: accountants receive tasks, work through them efficiently, and hand the resolved data back to the user's accounting system.

---

## Role in the Pinto Platform

The Pinto Agent handles the vast majority of financial tasks without human involvement. However, when the Agent encounters data it cannot confidently resolve — a vendor mismatch, a missing mandatory field, an OCR ambiguity, an API failure — it creates a task and routes it to the Accountant Dashboard.

The accountant's job is to:
1. Review the flagged transaction with all available context
2. Correct or complete the data
3. Optionally ask the user for clarification via an Inquiry
4. Mark the task as resolved, triggering the completed entry to be posted to the accounting system

This hybrid model ensures that automation handles volume while human expertise handles edge cases, maintaining data accuracy across all accounts.

---

## Actors

| Actor | Role |
|---|---|
| AI Agent | Creates tasks when it detects blockers, errors, or issues it cannot self-resolve |
| Technical Team | Creates tasks for system-level issues requiring engineering or support review |
| Accountant | Receives all tasks, works through them, and sends Inquiries to users when needed |
| User (via Pinto App) | Never receives tasks directly; only receives Inquiries and responds through the Pinto App |

---

## Task Inbox

The task inbox is the accountant's primary workspace. It displays all tasks routed to the accountant team, organized by status.

### Inbox Tabs (Status Filters)

| Tab | Description |
|---|---|
| Inbox (Open) | New tasks waiting to be picked up |
| In Progress | Tasks the accountant has started working on |
| Waiting on User | Tasks paused while awaiting user response to an Inquiry |
| Blocked | Tasks that cannot proceed (accountant has flagged a blocker) |
| Done | Resolved tasks |

### Task Cards

Each task is displayed as a card or table row showing:
- Task title (short summary of the issue, e.g., "Invoice submission failed")
- Organization name (the business the task belongs to)
- Task type (Issue or Normal)
- A short description of the problem
- Footer actions: **Start**, **Cancel**

### Filters

Accountants can filter the inbox by:
- Organization name
- Creation date

---

## Working a Task

### Starting a Task

When an accountant clicks **Start** on a task card, the task status moves from `open` to `in_progress`, and the full task detail view opens. This detail view shows the complete financial form — either a Bill form or an Invoice form — pre-filled with whatever data the AI Agent was able to extract.

### Bill Form Fields

The bill form includes all fields needed to complete a supplier expense entry:

- Bill number
- Bill issue date and due date
- Total, subtotal, and tax amount
- Tax rate
- Currency
- Vendor (contact)
- Reference
- Tax amount type
- Payment object (account, amount, date) — if payment was not yet recorded
- Line items (description, price, quantity, unit price)

### Invoice Form Fields

The invoice form includes:

- Contact (customer)
- Currency
- Invoice date and due date
- Invoice number
- Tax amount type
- Reference
- Discount (account, amount, tax rate)
- Line items (account, description, quantity, unit amount, discount, item, tax rate)

### Completing a Task

Once the accountant has reviewed and corrected the data, they click **Resolve**. This moves the task to `done`, triggers the completed entry to be posted to the accounting system, and sends the user a WhatsApp notification: "We've completed your flow about [topic]. No action needed."

---

## Task Statuses & Transitions

| Status | Description |
|---|---|
| `open` | Task just created; waiting for accountant to pick up |
| `in_progress` | Accountant clicked Start and is actively working |
| `waiting_on_user` | Accountant has sent an Inquiry to the user; task is paused |
| `blocked` | Cannot proceed — accountant has flagged a blocker and recorded a reason |
| `done` | Resolved; transaction has been completed |
| `cancelled` | Task was withdrawn or identified as a duplicate |

**Transition flow (happy path):**
`open` → `in_progress` → *(optionally)* `waiting_on_user` → `in_progress` → `done`

**Alternative paths:**
- If the user does not provide a clear response: `blocked` (requires a reason to be recorded)
- If the task is invalid or duplicated: `cancelled`

---

## Sending an Inquiry to a User

When the accountant needs specific information from the business owner to complete a task, they can send an **Inquiry** — a structured question delivered via WhatsApp with a link to the Pinto App where the user responds.

### How to Send an Inquiry

While viewing a Bill or Invoice task in `in_progress`, the footer shows a **"Send inquiry to user"** button. Clicking it opens an Inquiry dialog where the accountant writes a message to the user.

### Inquiry Delivery

The user receives a WhatsApp message explaining the situation with a direct link to the Inquiry page in the Pinto App. Two example message formats:

**Confirmation-first** (high confidence, quick verify):
> "Action needed to finish your request. We found a mismatch for your vendor: 'Acmee Ltd.' vs 'Acme Ltd.' — Is Acme Ltd. correct? 👉 Open Inquiry"

**Edit-first** (low confidence, user must correct):
> "Help us finish your request. We couldn't confirm your vendor name. Current value: Acmee Ltd. 👉 Open Inquiry to confirm or correct it."

### After the User Responds

- The task automatically moves from `waiting_on_user` back to `in_progress`
- The accountant receives a notification linked to the same parent task
- The accountant reviews the user's response and proceeds to resolve the task

---

## Notifications

### Receiving a New Task
When a new task is created for the accountant team, a notification appears in the Retool inbox. It includes:
- Task title
- Organization name
- Task type (Issue or Normal)
- A short description
- A click action that opens the Task Details view and sets the status to `in_progress`

### Receiving a User's Inquiry Response
When a user submits a response to an Inquiry, the accountant receives a notification linked to the original parent task, signaling that the task is ready to resume.

---

## Design Principles

The Accountant Dashboard is designed around operational efficiency:

**Structured task queue** — Tasks are organized by status so accountants always know what is waiting, what is in progress, and what is blocked.

**Pre-filled context** — Every task arrives with all the data the AI extracted, reducing the amount of manual lookup needed.

**Tight user loop** — When user input is needed, the Inquiry system provides a clean, tracked mechanism with full status visibility on both sides.

**Audit trail** — Task status transitions, inquiry exchanges, and resolutions are all recorded, providing a complete history of how each transaction was handled.
