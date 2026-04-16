---
title: "8. Constraints & Dependencies"
publish: true
---

# Constraints & Dependencies

## External API Constraints

### Wafeq

| Constraint | Detail |
|---|---|
| Cost center name length | `name_en` and `name_ar` are limited to 200 characters. Pinto must enforce this at input time. |
| Cost center is line-item level | There is no document-level cost center field on invoices or bills. Pinto must apply the cost center ID to every line item individually. |
| Project P&L via Wafeq API | **[NEEDS INPUT]** — It is unconfirmed whether Wafeq's reporting API supports filtering by cost center ID natively, or whether Pinto must aggregate raw bill/invoice data from its own DB. |
| Wafeq cost center sync | Cost center must be created in Wafeq before it can be referenced on any line item. Project creation in Pinto must be atomic with Wafeq cost center creation. |
| Wafeq sandbox availability | Wafeq sandbox environment must be confirmed available for integration testing before QA begins. |

---

### Zoho Books

| Constraint | Detail |
|---|---|
| `project_id` is line-item level only | There is no document-level project field on Zoho invoices. All line items must carry the `project_id` individually. |
| Zoho project creation dependencies | **[NEEDS INPUT]** — Zoho Books Projects may require a customer association when creating a project entity. If so, Pinto cannot create a standalone cost-center-style project in Zoho, and an alternative tagging approach (e.g., Reporting Tags) may be needed. |
| Zoho bill line items with `project_id` | **[NEEDS INPUT]** — Confirm that Zoho Books bill (purchase) line items also accept `project_id`, not just invoice line items. |
| Zoho project P&L reporting | **[NEEDS INPUT]** — Confirm the Zoho Books API endpoint for pulling a project-level P&L report, and whether it requires the project to have a customer association. |

---

## Internal Technical Dependencies

| Dependency | Detail |
|---|---|
| Database migration | A `projects` table must be created and `project_id` nullable FK columns added to `bills` and `invoices` tables before any agent or frontend work can be tested end-to-end. |
| Backend API must ship before agent tools | The n8n tool workflows (`list_projects`, `create_project`, `assign_project`) all call backend REST endpoints. Backend endpoints must be deployed to dev before agent integration testing can begin. |
| Web App SDK update | The `@alpha.apps/pinto-web-app-sdk` auto-generated SDK (v1.2.9) will need to be regenerated to include new project endpoints before frontend work can be completed. |
| "General" project convention | The "General" catch-all must be defined as a system-level convention in the backend (not a real project record) so that P&L aggregation correctly groups untagged transactions without creating a polluted project entity. |

---

## Feature Dependencies (Other Backlog Items)

| Feature | Relationship |
|---|---|
| **Query accounting system data** (P1 backlog) | Project P&L reporting depends on the ability to read and aggregate transaction data from Wafeq/Zoho. If this feature is built first it will accelerate project reporting. |
| **Context memory across conversations** (P4 backlog) | Would enable the agent to remember the user's last-used project across sessions, reducing repetitive selection. Not a blocker for v1. |
| **Custom accounting queries** (P1 backlog) | Would allow users to ask "Show me all expenses tagged to Project X from vendor Y" — a natural extension of cost center tagging. Not a blocker for v1. |
| **Billable expense tracking** (Professional Services gap) | Cost center tagging is the foundation for distinguishing billable vs. non-billable expenses per project. This feature enables that gap to be closed in a future iteration. |

---

## Timeline Considerations

| Item | Note |
|---|---|
| Zoho API investigation | The Zoho project creation dependency must be resolved **before backend development begins** — it may change the integration approach entirely. |
| Wafeq reporting API | Must be confirmed before the Reporting Agent tool (`generate_project_pnl_report`) can be finalised. |
| Migration safety | Adding nullable FK columns to `bills` and `invoices` tables must be carefully tested — these are high-volume production tables. |
| Agent prompt changes | Changes to Expense Agent and Invoice Agent prompts carry regression risk. Prompt testing should run in parallel with backend development, not after. |

---

## Known Risks

| Risk | Severity | Mitigation |
|---|---|---|
| Zoho does not support standalone project creation (requires customer) | High | Investigate early; fall back to Zoho Reporting Tags if needed |
| Wafeq reporting API does not support cost center filtering | Medium | Pinto can aggregate from its own DB — more work but feasible |
| Agent prompt changes degrade existing flows | Medium | Run full regression on Expense and Invoice agent flows after every prompt change |
| Users skip tagging → low-quality data | Low (by design) | Tagging is optional; "General" is an acceptable fallback; adoption improves over time |
| Large orgs with many line items per invoice → Wafeq payload size | Low | Each line item individually carries cost center ID — test with invoices of 20+ line items |
