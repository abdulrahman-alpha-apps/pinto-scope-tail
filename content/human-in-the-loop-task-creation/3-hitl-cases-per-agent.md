---
title: "3. HITL Cases per Agent"
publish: true
---

# HITL Cases per Agent

This document maps every Human-in-the-Loop escalation case to the agent that owns it. For each case: what triggers it, what type of task it creates, and what the accountant needs to do.

---

## How to Read This Document

| Column | Meaning |
|---|---|
| Case ID | Unique identifier used in the request body `errorCode` |
| Trigger | The condition inside the agent that causes escalation |
| Category | `issue` / `normal_task` / `error` / `user_objection` |
| What the accountant does | The action required to resolve this task |

---

## Router Agent

The Router is the entry point for every message. HITL here means the system could not even begin processing.

| Case ID | Trigger | Category | What the accountant does |
|---|---|---|---|
| `ROUTER_UNRESOLVABLE_INTENT` | Agent loops or produces an intent it cannot route after retries | `normal_task` | Review the raw message, determine what the user wanted, and complete the action manually or re-route |
| `ROUTER_PIPELINE_CRASH` | Unhandled exception in the Router workflow before any agent is reached | `error` | Investigate the error, replay the message if recoverable, or contact the user for a resend |
| `ROUTER_TRN_SETUP_FAILED` | VAT/TRN configuration branch fails — TRN cannot be validated or created in Wafeq/Zoho | `issue` | Manually configure the TRN in the accounting system and confirm setup with the user |

**Fallback tool status:** `creating_a_task1` node exists in the Router diagram — needs to be wired to defined conditions.

---

## Expense Agent

Handles receipt/bill processing via OCR and manual conversation. Has an existing `Fallback dummy` tool that already calls the task API.

| Case ID | Trigger | Category | What the accountant does |
|---|---|---|---|
| `EXPENSE_OCR_FIELDS_MISSING` | OCR ran 3 times but mandatory fields (vendor, amount, date) are still missing | `issue` | Open the attached document, manually extract the missing fields, and complete the bill entry |
| `EXPENSE_VENDOR_MATCH_FAILED` | Vendor name found but no contact match reached sufficient confidence after user prompts | `issue` | Review the vendor name and candidates, select or create the correct contact, then post the bill |
| `EXPENSE_TRN_INVALID` | TRN on the document fails FTA format check (not 15 digits starting with 1) | `issue` | Verify the correct TRN with the user or vendor, set the right tax treatment, and post the bill |
| `EXPENSE_ACCOUNT_UNRESOLVED` | Chart of Accounts category could not be confirmed by the user within the retry window | `issue` | Choose the correct account category and complete the expense entry |
| `EXPENSE_POST_API_FAILURE` | All data confirmed but the Wafeq/Zoho API returned an error during bill posting | `error` | Use the pre-filled data in the task to manually post the bill in the accounting system |
| `EXPENSE_USER_VAT_OBJECTION` | User explicitly disputes the VAT treatment the agent applied (e.g., reclaimable vs non-reclaimable) | `user_objection` | Review the correct VAT classification, update the bill's tax treatment, and post |
| `EXPENSE_USER_CATEGORY_OBJECTION` | User disputes the account category or vendor the agent used | `user_objection` | Correct the category or vendor on the existing record and confirm with the user |

**Fallback tool status:** `Fallback dummy` already implemented. Request body standardisation needed.

---

## Invoice Agent

Handles invoice creation, customer and item matching, and invoice delivery. Has an `exit` tool for clean conversation closure but no HITL fallback tool yet.

| Case ID | Trigger | Category | What the accountant does |
|---|---|---|---|
| `INVOICE_CUSTOMER_CREATE_FAILED` | No customer match found and the Create Customer API call failed | `issue` | Create the customer contact manually in the accounting system, then create and send the invoice |
| `INVOICE_ITEM_UNRESOLVED` | One or more line items could not be matched or created in the chart of accounts | `issue` | Resolve the unmatched items, complete the line item list, and post the invoice |
| `INVOICE_POST_API_FAILURE` | All invoice data confirmed but the accounting system API returned an error during submission | `error` | Use the pre-filled data to manually post the invoice, then notify the user |
| `INVOICE_CONVERSATION_TIMEOUT` | Invoice flow went idle (Timeout Notifier fired, user did not respond to reminder) | `normal_task` | Contact the user via Inquiry to collect any missing fields, then complete and send the invoice |
| `INVOICE_USER_OBJECTION` | User disputes line items, prices, quantities, or VAT on a confirmed or posted invoice | `user_objection` | Review and correct the invoice details per the user's stated change, update or void-reissue as needed |

**Fallback tool status:** New tool needs to be built. `exit` tool is not a substitute — it closes the conversation without creating a task.

---

## Payment Agent

Handles payment recording and bank statement reconciliation. Has an existing `Payment Fallback` tool.

| Case ID | Trigger | Category | What the accountant does |
|---|---|---|---|
| `PAYMENT_CONTACT_UNRESOLVED` | Match Contact and Create Contact both failed — payer/payee cannot be linked to a Wafeq/Zoho contact | `issue` | Locate or create the correct contact, then allocate the payment |
| `PAYMENT_INVOICE_NOT_FOUND` | Get Invoice/Bill Details finds no open match or finds multiple ambiguous matches for the payment | `issue` | Identify the correct invoice or bill and apply the payment manually |
| `PAYMENT_BANK_STATEMENT_PARSE_FAILED` | Uploaded CSV or PDF bank statement could not be parsed into valid payment rows | `issue` | Download the attached file, manually extract the payment rows, and record each payment |
| `PAYMENT_POST_API_FAILURE` | All payment details confirmed but the Wafeq/Zoho API returned an error during posting | `error` | Use the pre-filled data to manually post the payment in the accounting system |
| `PAYMENT_USER_OBJECTION` | User disputes the payment amount, date, method, or the invoice it was allocated to | `user_objection` | Correct the payment record or re-allocate per the user's stated preference |

**Fallback tool status:** `Payment Fallback` already implemented. Request body standardisation needed.

---

## Reporting Agent

Handles financial report generation (P&L, balance sheet, cash flow, statement of account). No fallback or task-creation path currently exists.

| Case ID | Trigger | Category | What the accountant does |
|---|---|---|---|
| `REPORT_CONTACT_UNRESOLVED` | User requests a Statement of Account but the contact name could not be matched in the accounting system | `issue` | Identify the correct contact, generate the Statement of Account manually, and send it to the user via WhatsApp or Inquiry |
| `REPORT_TYPE_UNSUPPORTED` | User requests a report type or customisation level the agent cannot generate automatically | `normal_task` | Prepare the requested report manually (or advise the user what is available), and deliver it |
| `REPORT_API_FAILURE` | Generate Reports or Generate Statement of Account call returns a non-recoverable API error | `error` | Generate the report manually from the accounting system and deliver it to the user |

**Fallback tool status:** No fallback tool exists. New tool needs to be built.

---

## Onboarding Agent

Guides new users through connecting their accounting system (Wafeq or Zoho) and configuring their setup. No task-creation path currently exists.

| Case ID | Trigger | Category | What the accountant does |
|---|---|---|---|
| `ONBOARDING_OAUTH_FAILED` | User attempts the OAuth flow but no valid token is received — accounting system is not connected | `issue` | Contact the user to retry the connection, or assist them directly in linking their accounting system |
| `ONBOARDING_TAX_CONFIG_INVALID` | After connecting, Pinto detects that the accounting system has incorrect tax settings (wrong rates, wrong account types, non-standard VAT groups) | `issue` | Log into the accounting system, correct the tax configuration, and confirm to the user that setup is complete |
| `ONBOARDING_PROFILE_UPDATE_FAILED` | Update User Info API call fails — the organization profile remains incomplete | `error` | Complete the profile update manually via the backend or dashboard, then confirm with the user |

**Fallback tool status:** No fallback tool exists. New tool needs to be built.

---

## Summary — HITL Cases by Agent

| Agent | Total Cases | issue | normal_task | error | user_objection |
|---|---|---|---|---|---|
| Router | 3 | 1 | 1 | 1 | 0 |
| Expense | 7 | 4 | 0 | 1 | 2 |
| Invoice | 5 | 2 | 1 | 1 | 1 |
| Payment | 5 | 3 | 0 | 1 | 1 |
| Reporting | 3 | 1 | 1 | 1 | 0 |
| Onboarding | 3 | 2 | 0 | 1 | 0 |
| **Total** | **26** | **13** | **3** | **6** | **4** |

---

## Implementation Status

| Agent | Fallback Tool | Action Needed |
|---|---|---|
| Router | `creating_a_task1` node exists but unwired | Wire to the 3 defined conditions above |
| Expense | `Fallback dummy` — exists | Standardise request body per case |
| Invoice | None | Build new HITL fallback tool |
| Payment | `Payment Fallback` — exists | Standardise request body per case |
| Reporting | None | Build new HITL fallback tool |
| Onboarding | None | Build new HITL fallback tool |
