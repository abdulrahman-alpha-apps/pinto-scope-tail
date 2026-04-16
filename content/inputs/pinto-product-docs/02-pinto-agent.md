---
title: "02-pinto-agent"
publish: true
---

# The Pinto Agent

## What is the Pinto Agent?

The Pinto Agent is the AI-powered conversational engine at the heart of the Pinto platform. It operates entirely through WhatsApp, acting as a virtual accountant that understands natural language, processes financial documents, and executes accounting tasks automatically. For most business owners, the Agent is Pinto — it is the primary and often only interface they ever use.

The Agent is designed around one core principle: the user should never need to understand accounting to manage their finances.

---

## How the Agent Works

When a user sends a message on WhatsApp — whether text, voice, an image, or a PDF — the Agent processes it through a multi-step pipeline:

1. **Intent recognition** — The Router Agent identifies what the user wants to do (record an expense, create an invoice, make a payment, request a report, etc.)
2. **Document processing** — If a document is attached, the OCR pipeline extracts structured financial data (amounts, vendors, dates, VAT, line items)
3. **Data validation** — The relevant specialist agent validates the extracted data, checks for missing fields, and confirms classifications
4. **Smart follow-up** — If information is ambiguous or missing, the Agent asks clarifying questions in a natural, conversational way
5. **Accounting system update** — Once confirmed, the Agent creates or updates the relevant entry in Wafeq or Zoho Books in real time
6. **Escalation if needed** — If the Agent fails to resolve an issue after multiple attempts, it automatically creates a task for the accountant team

---

## Agent Architecture

Pinto uses a multi-agent architecture where each agent specializes in a specific accounting domain. A central Router Agent directs incoming requests to the correct specialist.

### Router Agent
The Router Agent is the entry point for every user message. It interprets the user's intent and routes the request to the appropriate specialist agent. It also handles routing errors and escalates to the accountant dashboard when an entire workflow fails.

### Expense Agent
Handles all expense-related tasks:
- Accepts receipts and bills via photo, PDF, or manual text input
- Runs OCR extraction to identify vendor, amount, date, VAT, TRN, and line items
- Validates the supplier TRN against the FTA database
- Matches the vendor against existing contacts or creates a new one
- Suggests the correct Chart of Accounts category
- Detects the expense type: Paid Bill, Unpaid Invoice, Prepaid, etc.
- Supports bulk uploads (multiple receipts in one conversation)
- Records the expense in the accounting system

### Invoice Agent
Handles invoice creation and management:
- Accepts invoice requests via conversational text ("Invoice ABC Company for AED 5,000")
- Matches existing customers or creates new contact records
- Applies 5% VAT automatically and calculates totals
- Generates a professional PDF invoice
- Supports multi-item invoices within a single conversation
- Tracks due dates and payment terms
- Sends the invoice to the customer directly via WhatsApp

### Payment Agent
Handles payment recording and reconciliation:
- Records payments against open invoices or bills
- Handles partial payments and displays outstanding balances
- Detects and stores overpayments as advance credit, which is automatically applied to future invoices
- Supports bulk payment allocation (e.g., a single payment matched to the oldest outstanding invoices)
- Tracks payment method: Cash, Card, Bank Transfer
- Supports bank statement reconciliation

### Reporting Agent
Handles financial reporting and queries:
- Responds to natural language report requests ("Show me my profit last month", "What does ABC Company owe me?")
- Generates Profit & Loss statements, Balance Sheets, Cash Flow reports, and Statements of Account
- Delivers reports as PDF via WhatsApp
- Supports flexible date ranges through conversational input

### Onboarding Agent
Handles new user setup:
- Guides the user through connecting their accounting system (Wafeq or Zoho Books) via OAuth
- Configures Chart of Accounts based on business type and industry
- Imports existing customer and vendor data
- Configures VAT registration and tax settings automatically from the connected accounting system

---

## Key Capabilities

### OCR & Document Intelligence
The Agent can extract structured financial data from photographs and PDF documents. It identifies:
- Invoice/bill number, date, due date
- Vendor name and TRN
- Line items, quantities, unit prices
- Subtotals, VAT amounts, and totals
- Currency and payment reference

Extracted data is validated and normalized before being passed to the accounting system.

### VAT & TRN Handling
- Automatically applies 5% UAE VAT to applicable transactions
- Distinguishes between standard-rated (5%) and zero-rated (0%) transactions, such as exports
- Validates supplier TRNs against the FTA database in real time
- Maintains VAT-ready records suitable for FTA audit

### Multi-Currency Support
The Agent handles transactions in multiple currencies, converting and recording them correctly within the accounting system.

### Voice-to-Text
Users can send voice notes on WhatsApp and the Agent transcribes and processes them as text instructions.

### Memory & Context
The Agent retains context within a conversation session, allowing users to reference previous messages naturally (e.g., "Actually, make that AED 3,000 instead").

---

## Task Creation & Escalation

The Agent does not silently fail. When it encounters a situation it cannot resolve automatically, it creates a structured task and escalates it to the accountant team. This ensures no transaction is lost or left in an unknown state.

### When the Agent Creates a Task
- After **three failed attempts** to resolve an issue automatically (e.g., a vendor name mismatch that cannot be confirmed)
- When an **unexpected processing error** occurs during any workflow
- When a **conversation thread remains open and idle** for approximately one hour (the backend runs a scheduled job every 24 hours to catch these)

### Task Types
The Agent categorizes every task it creates as one of two types:

**Issue Task** — Created when data is incorrect, conflicting, or incomplete in a way that blocks automation. Examples: names don't match, mandatory fields are missing, prior automated correction attempts have failed. The accountant must review and fix the data.

**Normal Task** — Created when the user simply needs to complete a routine action (upload a document, confirm a value, finish a started flow) and the conversation can be tracked and closed cleanly.

### What Happens After a Task is Created
1. The task is sent to the Accountant Dashboard (Retool)
2. The user receives a WhatsApp notification informing them that their request is being reviewed
3. Once the accountant resolves the task, the user is notified via WhatsApp: "We've completed your flow. No action needed."
4. If the accountant needs information from the user to complete the task, they send an **Inquiry** — a WhatsApp message with a link to the user's web app where they can respond

---

## Accounting System Integration

The Agent integrates with accounting platforms via secure OAuth, meaning users never share their passwords. Supported platforms:

- **Wafeq** — A UAE-native accounting platform with strong VAT compliance features
- **Zoho Books** — A widely used SME accounting solution

Every action taken by the Agent — recording an expense, creating an invoice, logging a payment — is reflected in the accounting system in real time. The Agent does not replace the accounting software; it makes interacting with it effortless.

---

## Error Handling Summary

| Error Type | Agent Behaviour |
|---|---|
| OCR extraction fails | Asks user to resend a clearer image; escalates after 3 attempts |
| Vendor not found / mismatch | Prompts user to confirm; escalates if unresolved |
| Chart of Accounts not matched | Suggests closest match; asks user to confirm |
| Payment account not identified | Asks user to specify; escalates if unresolved |
| Accounting system API failure | Creates a task for the technical team |
| Conversation idle > 1 hour | Scheduled job sends a reminder; creates task if user chooses to continue |
