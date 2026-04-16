---
title: "10. Accounting Logic Summary"
publish: true
---

# Accounting Logic Summary — Cost Center / Project-Level P&L Tracking

## Purpose of This Document
This document captures the accounting-specific considerations for the Cost Center feature. It is written for the accountant team and anyone reviewing the financial logic behind how project tagging affects the books.

---

## Core Accounting Concept

A **cost center** (called "project" in Pinto's user-facing language) is an accounting classification that allows a business to group revenues and costs by a specific activity, job, or client engagement — separate from the overall company P&L.

Cost centers do **not** affect:
- The chart of accounts
- VAT treatment or tax codes
- Payment terms or due dates
- Invoice or bill totals
- Any accounting entry in the general ledger

They **only** add a classification tag to each line item. The accounting entries themselves are identical with or without a cost center tag. The cost center tag enables filtered reporting — nothing else.

---

## How Project Tagging Works in the Books

### Expenses (Bills)
When a user tags an expense to a project:

```
DR  Raw Materials Expense   AED 3,200   [Cost Center: Alpha Fitout]
CR  Accounts Payable                    AED 3,200
```

The journal entry is identical to an untagged expense. The cost center annotation sits on the line item in Wafeq — it does not create a separate account or sub-account.

### Invoices (Revenue)
When a user tags an invoice to a project:

```
DR  Accounts Receivable     AED 45,000
CR  Revenue / Sales         AED 45,000  [Cost Center: Alpha Fitout]
```

Again, the journal entry is unchanged. The cost center label is attached at the line item level in Wafeq.

---

## Project P&L Calculation

Pinto calculates project P&L by aggregating tagged line items:

| Line | Calculation |
|---|---|
| **Project Revenue** | SUM of all invoice line item amounts where `cost_center = Project X` |
| **Project Costs** | SUM of all bill/expense line item amounts where `cost_center = Project X` |
| **Gross Profit** | Revenue − Costs |
| **Margin %** | (Gross Profit ÷ Revenue) × 100 |

**Important:** This is a **gross margin** calculation, not a net profit calculation. Overhead costs that are not tagged to a specific project (e.g., rent, utilities, salaries) will land in "General" and are **not** allocated across projects. Users should understand that project profitability in Pinto represents direct project costs only, not fully-loaded profitability.

This should be clearly communicated in the UI and in any WhatsApp report summary.

---

## The "General" Project

All untagged transactions — both before this feature ships and any transactions the user chooses not to tag — are grouped under **"General"**.

General is not a real project in the accounting system. It is a virtual grouping in Pinto's reporting layer. It represents:
- Overhead costs (rent, subscriptions, salaries if recorded as expenses)
- Historical transactions from before cost center tagging was available
- Any transaction the user explicitly skipped tagging

The General P&L will typically show costs greater than revenue (since overhead is rarely directly billed to a project). This is expected and correct.

---

## VAT Considerations

Cost center tagging has **no effect on VAT**. The following rules are unchanged:

| Rule | Status |
|---|---|
| 5% standard VAT rate on applicable transactions | Unchanged |
| Zero-rated transactions (exports, certain services) | Unchanged |
| TRN validation on vendor bills | Unchanged |
| VAT recorded on bill/invoice line items | Unchanged |

If a bill has multiple line items with different VAT rates, each line item carries its own VAT — and each also carries the same cost center tag if the expense is assigned to a project. There is no scenario where the cost center tag changes the VAT treatment.

---

## Multi-Currency Considerations

When a project contains transactions in multiple currencies (e.g., a USD invoice from an international client and AED expenses):

- Each transaction is stored in its original currency in Wafeq/Zoho
- Pinto's project P&L aggregation converts all amounts to AED using the **exchange rate recorded at transaction time**
- Exchange rates are not retrospectively adjusted
- This means the project P&L is always expressed in AED

**Accountant note:** If a user questions why their project P&L figures don't exactly match Wafeq's figures, the likely cause is currency conversion timing differences. This should be flagged in the accountant dashboard if it becomes a source of reconciliation queries.

---

## Partial Tagging Risk

Because tagging is optional, a project may have:
- Some expenses tagged, some not (some fall into General)
- Some invoice line items tagged, some not

This creates an **understated project P&L** — the project appears less profitable or more costly than it actually is, because not all related transactions are tagged.

**Accountant guidance:** If a client is asking why a project's P&L looks wrong, the first check should be whether all related expenses and invoices were tagged correctly. The accountant can review untagged transactions in the "General" bucket to identify candidates for re-tagging.

**Note for v1:** Retroactive re-tagging (assigning a project to an already-posted transaction) is **not supported in v1**. This is a v2 enhancement. Accountants should note this limitation when advising clients.

---

## Accountant Dashboard Implications

### Bill Task Form
When an accountant reviews and completes a bill task that the user tagged to a project:
- The project name should be visible in the bill form (read-only, informational)
- The accountant must not remove the project tag when resolving the task
- When the accountant posts the bill to Wafeq, the `cost_center` field must be included on each line item

### Invoice Task Form
Same principle — project tag is preserved through the accountant's resolution flow.

### No Accountant-Initiated Tagging (v1)
Accountants cannot tag transactions to projects from the Accountant Dashboard in v1. Only the user can assign a project (via WhatsApp). This is a v2 consideration.

---

## Reconciliation Notes

| Scenario | Accounting Impact |
|---|---|
| User tags expense to wrong project | Books are correct (correct account, correct VAT); only the cost center classification is wrong. No ledger correction needed — cost center is a reporting tag only. |
| User closes a project | Historical tagged transactions retain the cost center in Wafeq. Reporting still works for closed projects. No reversal or correction needed. |
| Wafeq cost center deleted externally | Pinto's DB retains `wafeq_cost_center_id` but the cost center no longer exists in Wafeq. P&L aggregation falls back to Pinto's own DB. Flag for reconciliation review. |

---

## Summary for Accountants

- Cost centers are **classification labels only** — they do not affect any accounting entries, VAT, or payment terms.
- Project P&L shows **direct project costs and revenues** — not fully-loaded profitability.
- Untagged transactions land in **General** — this is by design.
- **Tagging is optional** — partial tagging produces understated project P&L; educate clients on this.
- **Retroactive re-tagging is not available in v1** — transactions cannot be re-assigned to a project after they are posted.
- The correct place to raise client questions about project P&L discrepancies is the **inquiry system** (send an inquiry to the user asking them to confirm which project a transaction belongs to).
