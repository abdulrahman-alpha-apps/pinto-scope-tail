---
title: "3. HITL Cases per Agent"
publish: true
---

# HITL Cases per Agent — MVP

Each agent has at most 4 task types. Cases like "vendor not matched", "account not resolved", and "OCR fields missing" all collapse into one **Data Failure** task — the accountant reviews and fills in whatever is missing.

---

## Category Mapping

This is what you type in the `category` field of the API request body:

| Task Type | `category` value | Meaning |
|---|---|---|
| Data Failure | `"issue"` | Agent couldn't resolve required data — accountant must review and correct |
| System Failure | `"error"` | Technical/API error — posting failed after data was confirmed |
| User Objection | `"user_objection"` | User explicitly disagrees with what the agent did |
| Timeout | `"normal_task"` | Conversation went idle — accountant follows up to complete |
| Setup Failure | `"issue"` | Onboarding config is broken — accountant must fix |

> `user_objection` is a new value — requires a backend schema update before it can be used.

---

## Router Agent

| Task | Trigger | `category` |
|---|---|---|
| Routing failed | Intent could not be classified or pipeline crashed | `"error"` |

---

## Expense Agent

| Task                               | Trigger                                                             | `category`         |
| ---------------------------------- | ------------------------------------------------------------------- | ------------------ |
| Expense data could not be resolved | OCR failed, vendor not matched, account not resolved, TRN invalid   | `"issue"`          |
| Expense could not be posted        | Wafeq/Zoho API error after data was confirmed                       | `"error"`          |
| User disputes expense              | User objects to VAT treatment, category, or vendor applied by agent | `"user_objection"` |

---

## Invoice Agent

| Task                               | Trigger                                                     | `category`         |
| ---------------------------------- | ----------------------------------------------------------- | ------------------ |
| Invoice data could not be resolved | Customer not found, line items not matched                  | `"issue"`          |
| Invoice could not be posted        | Wafeq/Zoho API error after data was confirmed               | `"error"`          |
| Invoice flow timed out             | Conversation went idle before invoice was completed         | `"normal_task"`    |
| User disputes invoice              | User objects to line items, prices, or VAT applied by agent | `"user_objection"` |

---

## Payment Agent

| Task | Trigger | `category` |
|---|---|---|
| Payment data could not be resolved | Contact not matched, invoice not found, bank statement not parsed | `"issue"` |
| Payment could not be posted | Wafeq/Zoho API error after data was confirmed | `"error"` |
| User disputes payment | User objects to amount, date, method, or invoice allocation | `"user_objection"` |

---

## Reporting Agent

| Task | Trigger | `category` |
|---|---|---|
| Report could not be generated | Contact not matched, unsupported report type, or API error | `"issue"` or `"error"` |

---

## Onboarding Agent

| Task | Trigger | `category` |
|---|---|---|
| Accounting system could not be connected | OAuth failed or tax configuration is invalid | `"issue"` |
| Profile setup failed | User info update API call failed | `"error"` |

---

## Summary

| Agent | `"issue"` | `"error"` | `"user_objection"` | `"normal_task"` |
|---|---|---|---|---|
| Router | — | 1 | — | — |
| Expense | 1 | 1 | 1 | — |
| Invoice | 1 | 1 | 1 | 1 |
| Payment | 1 | 1 | 1 | — |
| Reporting | 1 | — | — | — |
| Onboarding | 1 | 1 | — | — |
| **Total** | **5** | **5** | **3** | **1** |
