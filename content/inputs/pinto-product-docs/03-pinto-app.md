---
title: "03-pinto-app"
publish: true
---

# The Pinto App (User Web Application)

## What is the Pinto App?

The Pinto App is a web application for business users — the same owners and operators who interact with Pinto through WhatsApp. While the AI Agent on WhatsApp handles the majority of day-to-day tasks, the Pinto App serves as a structured, visual interface for managing open issues, reviewing invoices and expenses, responding to accountant inquiries, and monitoring the overall financial health of the business through a reporting dashboard.

The app is not a replacement for the WhatsApp experience. It is a complementary layer that serves two distinct purposes: it activates when something requires the user's attention (tasks, inquiries, pending items), and it provides continuous financial visibility through charts, summaries, and reports that are always available without needing to start a conversation.

---

## Authentication

Users log in via their WhatsApp phone number. No password is required.

**Login Flow:**
1. User opens the Pinto App — either via a direct link or from a link sent to them via WhatsApp
2. The system checks if the user is already authenticated
3. If authenticated, the user is taken directly to their tasks dashboard (or the specific task detail if they followed a WhatsApp link)
4. If not authenticated, the user enters their WhatsApp phone number (often auto-filled), receives a one-time password (OTP) via WhatsApp, enters it, and is logged in
5. If the user is new, an account is created automatically upon OTP validation

This login method ensures a frictionless experience — the same WhatsApp number used with the Agent is the single identity across the entire Pinto platform.

---

## Home Page

Upon login, the user lands on their home page, which displays a clean list of all their open tasks. The goal of the home page is immediate clarity: the user should know at a glance what needs their attention.

From the home page, users can:
- See all open tasks in a prioritized list
- Resolve a task directly or tap into it for more detail
- Navigate to invoices or expenses for a broader financial view

---

## Tasks (Issues)

Tasks are the central feature of the Pinto App. A task is an issue card created either by the AI Agent (when it encounters something it cannot resolve automatically) or by an accountant (when they need user input to complete a financial entry).

### What a Task Represents

Tasks surface when something in a financial flow requires the user's confirmation or action. Examples include:

- A receipt where the vendor name could not be confirmed
- An invoice with missing line item details
- A bill where the OCR data extraction produced ambiguous results
- An expense that was uploaded but not completed

### Task Types

**Issue Task** — Raised when data is incorrect, conflicting, or incomplete. The user must review the information and provide or confirm the correct data before the transaction can be completed.

**Normal Task** — A routine action item. The user needs to complete a step they started (e.g., finish submitting an expense, upload a missing document).

### Task Attributes

Each task card displays:
- A short, clear title describing the issue (e.g., "Vendor name mismatch — Bill #9082")
- The task type (Issue or Normal)
- The related document type (Invoice or Bill)
- A description of the problem and what the user needs to do
- The current status
- Pre-filled data from the AI where available, so the user is not starting from scratch

### Task Statuses

| Status | Meaning |
|---|---|
| Open | Task is waiting for user action |
| Resolved | User has submitted their response |

### Resolving a Task

Users resolve tasks directly within the app by reviewing the pre-filled data, correcting or confirming the relevant fields, and submitting. Once submitted, the accountant receives a notification and resumes processing the transaction.

### Filters

Users can filter their task list by:
- Status (Open, Resolved)
- Task type (Issue, Normal)

---

## Inquiries

An inquiry is a specific type of task sent by the accountant when they need the user to answer a question or confirm a piece of data in order to complete a financial entry. It differs from a standard task in that it originates from a human accountant rather than the AI Agent.

**Inquiry Delivery:**
1. The accountant sends the inquiry from the Accountant Dashboard
2. The user receives a WhatsApp message with a brief explanation and a link
3. The link opens the Inquiry page in the Pinto App
4. The user reads the question, provides their response, and submits

**Example WhatsApp Message:**
> "Action needed to finish your request. We found a mismatch for your vendor: 'Acmee Ltd.' vs 'Acme Ltd.' — is 'Acme Ltd.' correct? 👉 Open Inquiry"

After submission, the user sees a confirmation: "Thanks! We've received your response. We'll finish processing shortly." The accountant is automatically notified and resumes the task.

---

## Invoice Management

The Pinto App gives users a structured view of all their invoices.

Users can:
- Browse the full list of invoices with key details visible at a glance
- Click into any invoice to see its full details

**Filter Options:**
- Contact name
- Invoice date (from / to)
- Due date (from / to)
- Invoice status (e.g., paid, unpaid, overdue)
- Sent status (Sent, Unsent, All)
- Invoice number

This view is particularly useful for tracking outstanding receivables and reviewing the history of a specific customer's invoices.

---

## Expense Management

The Pinto App provides a view of all uploaded expenses, including those that are fully processed and those still pending completion.

Users can:
- View all expenses with their status
- Continue processing a pending expense that was started but not finished
- Ignore an expense if it should not be recorded

**Filter Options:**
- Vendor name
- Upload date
- Status

The ability to "continue" a pending expense is an important recovery path — if a user started recording an expense on WhatsApp but the conversation was interrupted or timed out, they can pick up from where they left off directly in the app.

---

## Reporting Dashboard

The Pinto App includes a dedicated reporting dashboard that gives business owners a real-time visual overview of their financial health — all in one place, without needing to ask the Agent or wait for a report to be delivered via WhatsApp.

### What the Dashboard Shows

The reporting dashboard aggregates data from the user's connected accounting system and presents it through charts, summaries, and key metrics. It is designed to answer the most common questions a business owner has at a glance:

- How is my business performing this month?
- Am I profitable compared to last month?
- Where is my money going?
- Who owes me money?
- What is my cash position?

### Dashboard Content

**Financial Summaries** — Key figures such as total revenue, total expenses, and net profit for the current period, with comparisons to prior periods.

**Profit & Loss Overview** — A visual breakdown of income versus expenses, showing whether the business is running at a profit or loss over a selected time range.

**Cash Flow** — A view of cash inflows and outflows, helping owners understand the movement of money through the business.

**Expense Breakdown** — Charts showing spending by category, vendor, or time period, making it easy to spot where costs are concentrated.

**Receivables Summary** — Outstanding invoices and amounts owed by customers, giving a clear picture of what is yet to be collected.

**Recent Activity** — A feed of the latest transactions processed through Pinto, keeping the user informed of what has been recorded.

### Time Range & Filtering

Users can adjust the reporting period to view data by day, week, month, quarter, or custom date range, allowing them to track trends over time or drill into a specific period.

### Relationship to WhatsApp Reporting

The dashboard complements — rather than replaces — the WhatsApp-based reporting experience. Users who prefer conversational queries ("Show me last quarter's P&L") continue to get reports delivered as PDFs via WhatsApp. The dashboard provides continuous, always-on visibility for users who want to check in on their business at any time without starting a conversation.

---

## WhatsApp Reminders

The Pinto App is tightly integrated with WhatsApp for proactive notifications. Users receive reminders when:

- They have open tasks waiting for action
- There are pending invoices that require follow-up
- There are pending expenses that were not completed

Each reminder message includes a direct link to the relevant item in the Pinto App, allowing the user to go straight to the issue without navigating through menus.

---

## Design Principles

The Pinto App is built around three principles:

**Minimal friction** — Authentication requires only a WhatsApp OTP. Task resolution is designed to be completable in a few taps.

**Context continuity** — Tasks and inquiries always carry pre-filled data from the AI, so users are never asked to re-enter information the system already has.

**Action-oriented** — The home page surfaces tasks immediately. The app exists to resolve things, not to browse data.
