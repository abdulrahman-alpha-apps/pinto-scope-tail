---
title: "3. Function Scoping.a - Backend"
publish: true
---

# Function Scoping — a. Backend

## Functional Goal / Objective
The backend must introduce a `projects` (cost center) entity into Pinto's data model, expose CRUD APIs for project management, extend expense and invoice records to carry a `project_id` foreign key, and synchronise project tags to the correct fields in Wafeq and Zoho at write time. It must also support project-level P&L aggregation queries used by the Reporting Agent and the Web App dashboard.

---

## User Benefit Summary
Without backend project support, no surface (WhatsApp Agent, Web App, Accountant Dashboard) can tag, store, or report on project-level data. The backend is the single source of truth for which transactions belong to which project, and the bridge that carries that tag into the accounting system of record (Wafeq / Zoho).

---

## New Data Model

### `projects` Table (Pinto-owned)

| Column | Type | Notes |
|---|---|---|
| `id` | UUID | Primary key |
| `org_id` | UUID | FK to organisations table |
| `name` | VARCHAR(200) | Project display name |
| `status` | ENUM | `active`, `closed` |
| `wafeq_cost_center_id` | VARCHAR | Wafeq cost center ID (synced on creation) |
| `zoho_project_id` | VARCHAR | Zoho Books project ID (synced on creation) |
| `created_at` | TIMESTAMP | |
| `updated_at` | TIMESTAMP | |

### Extensions to Existing Tables

| Table | New Column | Type | Notes |
|---|---|---|---|
| `bills` / `expenses` | `project_id` | UUID (nullable) | FK to `projects` |
| `invoices` | `project_id` | UUID (nullable) | FK to `projects` |

---

## New API Endpoints

| Method | Path | Description |
|---|---|---|
| `POST` | `/projects` | Create a new project for an organisation |
| `GET` | `/projects` | List all projects for an organisation |
| `PATCH` | `/projects/:id` | Rename or update project status |
| `DELETE` | `/projects/:id` | Soft-delete / close a project |
| `GET` | `/projects/:id/pnl` | Return aggregated P&L summary for a project |

### Project Limit Enforcement
- Backend enforces a configurable `max_projects_per_org` setting (adjustable per org).
- Returns a `429`-style validation error with a user-friendly message when the limit is reached.

---

## Wafeq Integration

| Action | Wafeq API Behaviour |
|---|---|
| Create project | Call Wafeq `POST /cost_centers` with `name_en` (and optionally `name_ar`). Store returned `id` as `wafeq_cost_center_id`. |
| Tag expense (bill line item) | When creating/updating a bill via Wafeq API, include `cost_center: wafeq_cost_center_id` on each `BillLineItem` in the payload. |
| Tag invoice line item | Same pattern — include `cost_center` on each invoice line item where a project is assigned. |
| Close project | Call Wafeq `PATCH /cost_centers/{id}` with `is_active: false`. |
| Project P&L | [NEEDS INPUT] — Confirm whether Wafeq's reporting API supports filtering by cost center ID, or whether Pinto must aggregate from raw bill/invoice data filtered by cost center. |

**Wafeq field reference:**
- Cost center entity fields: `id` (string), `name_en` (string, max 200), `name_ar` (string), `is_active` (boolean)
- Line item field: `cost_center` (string — the cost center `id` value)

---

## Zoho Books Integration

| Action | Zoho Behaviour |
|---|---|
| Create project | [NEEDS INPUT] — Zoho Books has a Projects module. Confirm whether `POST /projects` in Zoho Books API creates a standalone project entity, and whether it requires a customer association. |
| Tag invoice line item | Include `project_id` (string) on each entry in the `line_items[]` array of the invoice payload. |
| Tag bill/expense line item | [NEEDS INPUT] — Confirm whether Zoho Bills API `line_items[]` also accepts `project_id`. |
| Project P&L | Zoho Books supports project-level reporting. [NEEDS INPUT] — Confirm the exact report endpoint and required parameters. |

**Zoho field reference:**
- Invoice line item field: `project_id` (string — Zoho project identifier)
- Tagging is **line-item level only** — there is no document-level project field on Zoho invoices.

---

## P&L Aggregation Logic

Pinto computes project P&L from its own database (not by querying Wafeq/Zoho live for every request):

```
Project Revenue  = SUM of invoice line item amounts where project_id = X
Project Costs    = SUM of bill/expense line item amounts where project_id = X
Net Profit       = Revenue - Costs
Margin %         = (Net Profit / Revenue) * 100
```

- Aggregation should support optional date range filtering.
- Synced after each write to keep Pinto's view consistent.
- "General" project is a virtual project (no `wafeq_cost_center_id` / `zoho_project_id`) — untagged transactions are grouped here at query time.

---

## Data Flow

```
WhatsApp → WATI → n8n (Expense/Invoice/Router Agent)
  → Backend POST /expenses or /invoices (with project_id in payload)
    → Pinto DB: bill/invoice record stored with project_id FK
    → Wafeq: bill line items created with cost_center field
    → Zoho: invoice/bill line items created with project_id field
      → Sync confirmed back to Pinto DB
```

---

## VAT / Currency / Multi-Tenant Considerations

| Consideration | Impact |
|---|---|
| VAT | Project tagging does not affect VAT treatment. Existing VAT logic is unchanged. |
| Currency | Project P&L aggregation must normalise multi-currency line items to the org's base currency (AED) using the exchange rate stored at transaction time. |
| Multi-tenant | `project_id` is always scoped to `org_id`. No cross-org data leakage possible. |

---

## Constraints

- Wafeq `name_en` field is limited to 200 characters — enforce this in Pinto's project name validation.
- Zoho project creation dependencies need confirmation — Zoho may require a customer to be associated with a project (not suitable for cost-center use).
- Historical transactions (before this feature ships) will not have `project_id` — they land in "General" by default.
- Project deletion must be soft-delete only — hard deletion would orphan historical accounting records in Wafeq/Zoho.

---

## Connections & Integrations

| Connects To | How |
|---|---|
| Expense Agent (n8n) | Receives `project_id` in the record expense tool payload |
| Invoice Agent (n8n) | Receives `project_id` in the create invoice tool payload |
| Reporting Agent (n8n) | Calls `GET /projects/:id/pnl` to fetch project P&L data |
| Web App | Reads project list and P&L summaries via backend API |
| Wafeq | Creates/updates cost center entities and tags line items |
| Zoho Books | Tags invoice/bill line items with `project_id` |
