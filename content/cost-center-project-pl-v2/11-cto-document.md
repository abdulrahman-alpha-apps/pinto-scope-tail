---
title: "11. CTO Document"
publish: true
---

# CTO Document — Cost Center / Project-Level P&L Tracking

## Executive Summary
This feature introduces project-level cost center tagging across Pinto's expense and invoice flows, with aggregated P&L reporting per project. It is a **P0 backlog item** that unlocks five high-priority segments simultaneously. Architecturally it is a well-contained addition — no new agent, no new top-level workflow, no changes to the accounting data model — but it requires careful integration work across three agents, two accounting providers, and a DB migration on production tables.

**Complexity: High**
**Risk: Medium** (gated on two Zoho API unknowns)
**Systems affected:** Backend (DB + API), n8n (3 agents, 5 new tool workflows), Web App (new route + components), Wafeq integration, Zoho integration

---

## Architecture Overview

```
WhatsApp User
    │
    ▼
WATI → n8n Router Agent (no change)
    │
    ├── Expense Agent (modified)
    │       ├── list_projects tool  ──────────────────────┐
    │       └── assign_project tool ─────────────────────┐│
    │                                                     ││
    ├── Invoice Agent (modified)                          ││
    │       ├── list_projects tool  ──────────────────────┤│
    │       └── assign_project tool ─────────────────────┤│
    │                                                     ││
    └── Reporting Agent (modified)                        ││
            ├── list_projects tool  ──────────────────────┤│
            └── generate_project_pnl_report tool ─────────┤│
                                                          ││
Pinto Backend (Node.js / AWS)  ◄──────────────────────────┘│
    │   ├── POST /projects          ◄───────────────────────┘
    │   ├── GET  /projects
    │   ├── PATCH /projects/:id
    │   ├── GET  /projects/:id/pnl
    │   └── PATCH /expenses/:id, /invoices/:id (project_id)
    │
    ├──► Wafeq API
    │       ├── POST /cost_centers       (on project create)
    │       ├── PATCH /cost_centers/:id  (on project close)
    │       └── cost_center field on BillLineItem / InvoiceLineItem
    │
    └──► Zoho Books API
            ├── POST /projects [NEEDS CONFIRMATION]
            └── project_id on line_items[]

Web App (Next.js 14 static export)
    └── /dashboard/projects  (new route, read-only)
```

---

## Database Changes

### New Table: `projects`

```sql
CREATE TABLE projects (
  id                    UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id                UUID NOT NULL REFERENCES organisations(id),
  name                  VARCHAR(200) NOT NULL,
  status                VARCHAR(20) NOT NULL DEFAULT 'active', -- 'active' | 'closed'
  wafeq_cost_center_id  VARCHAR(255),
  zoho_project_id       VARCHAR(255),
  created_at            TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at            TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_projects_org_id ON projects(org_id);
CREATE UNIQUE INDEX idx_projects_org_name ON projects(org_id, name);
```

### Migrations on Existing Tables

```sql
-- Add project_id FK to bills table
ALTER TABLE bills ADD COLUMN project_id UUID REFERENCES projects(id);

-- Add project_id FK to invoices table
ALTER TABLE invoices ADD COLUMN project_id UUID REFERENCES projects(id);
```

**Risk note:** `bills` and `invoices` are high-volume production tables. Migration must use `ADD COLUMN ... DEFAULT NULL` (no full table rewrite on Postgres). Verify row counts and test migration timing in staging before production deploy. Both columns are nullable — existing records are unaffected.

---

## New API Endpoints

| Method | Route | Auth | Description |
|---|---|---|---|
| `POST` | `/projects` | Org-scoped | Create project; creates Wafeq cost center + Zoho project atomically |
| `GET` | `/projects` | Org-scoped | List projects; supports `?status=active\|closed\|all` |
| `PATCH` | `/projects/:id` | Org-scoped | Rename or change status |
| `DELETE` | `/projects/:id` | Org-scoped | Soft-delete (sets `status = closed`) |
| `GET` | `/projects/:id/pnl` | Org-scoped | P&L aggregation; supports `?from=&to=` date params |
| `GET` | `/projects/pnl` | Org-scoped | All-projects P&L summary (for dashboard card) |

All endpoints are organisation-scoped — `org_id` from the auth context, never from the request body.

---

## P&L Query Design

The `GET /projects/:id/pnl` endpoint must be efficient. Recommended approach:

```sql
-- Revenue: sum of invoice line items tagged to this project
SELECT COALESCE(SUM(ili.amount * COALESCE(i.exchange_rate, 1)), 0) AS revenue
FROM invoice_line_items ili
JOIN invoices i ON i.id = ili.invoice_id
WHERE i.project_id = $1
  AND i.org_id = $2
  AND ($3::date IS NULL OR i.invoice_date >= $3)
  AND ($4::date IS NULL OR i.invoice_date <= $4);

-- Costs: sum of bill line items tagged to this project
SELECT COALESCE(SUM(bli.amount * COALESCE(b.exchange_rate, 1)), 0) AS costs
FROM bill_line_items bli
JOIN bills b ON b.id = bli.bill_id
WHERE b.project_id = $1
  AND b.org_id = $2
  AND ($3::date IS NULL OR b.bill_date >= $3)
  AND ($4::date IS NULL OR b.bill_date <= $4);
```

Index required:
```sql
CREATE INDEX idx_invoices_project_id ON invoices(project_id);
CREATE INDEX idx_bills_project_id ON bills(project_id);
```

"General" aggregation: same queries with `WHERE project_id IS NULL`.

---

## Wafeq Integration Detail

| Operation | Endpoint | Payload |
|---|---|---|
| Create cost center | `POST /cost_centers` | `{ name_en: string, name_ar?: string, is_active: true }` |
| Close cost center | `PATCH /cost_centers/{id}` | `{ is_active: false }` |
| Tag bill line item | Inside `POST /bills` payload | `line_items[n].cost_center = wafeq_cost_center_id` |
| Tag invoice line item | Inside `POST /invoices` payload | `line_items[n].cost_center = wafeq_cost_center_id` |

**Atomicity:** Project creation in Pinto DB and Wafeq cost center creation must be wrapped in a compensating transaction pattern — if the Wafeq call fails, roll back the Pinto DB record. If the Pinto DB write fails after a successful Wafeq call, store the `wafeq_cost_center_id` in a retry queue to be linked on next attempt.

---

## Zoho Books Integration — Unresolved Items

Two Zoho API questions must be resolved before backend development begins:

### Issue 1 — Project Creation Requires Customer Association
Zoho Books Projects are designed for customer-based project billing. Creating a standalone cost-center-style project without a customer may not be supported.

**If confirmed unsupported:** Use **Zoho Books Reporting Tags** as the alternative.
- Create a Reporting Tag category called "Project" on first use.
- Create a tag option per project (e.g., "Alpha Fitout" under "Project" category).
- Apply the tag option to each invoice/bill line item instead of a `project_id`.
- Zoho's reporting engine supports filtering by reporting tags — project P&L queries remain possible.

This fallback is clean architecturally — the Pinto backend simply routes to a different Zoho integration module depending on whether it's a project entity or a reporting tag.

### Issue 2 — Zoho Bill Line Items with `project_id`
`project_id` on invoice line items is confirmed. On bill (purchase) line items it is unconfirmed. Must test in Zoho sandbox before committing to this pattern for expense tagging.

**Action required:** Backend team to test both scenarios in Zoho sandbox before sprint kickoff.

---

## n8n Tool Workflows — New Additions

| Workflow Name | Type | HTTP Call | Used By |
|---|---|---|---|
| `list_projects` | Tool workflow | `GET /projects?org_id=&status=active` | Expense, Invoice, Reporting agents |
| `create_project` | Tool workflow | `POST /projects` | All agents (inline intent) |
| `assign_project` | Tool workflow | `PATCH /expenses/:id` or `/invoices/:id` | Expense, Invoice agents |
| `generate_project_pnl_report` | Tool workflow | `GET /projects/:id/pnl` | Reporting agent |
| `close_project` | Tool workflow | `PATCH /projects/:id` | Standalone project intent |

**Pattern:** All tools follow the existing HTTP-call-to-backend tool workflow pattern. No direct Wafeq/Zoho calls from n8n — all provider integration is handled by the backend.

---

## Agent Prompt Changes — Regression Risk

Three agent prompts must be updated. This is the highest regression risk in this feature.

| Agent | Change | Risk |
|---|---|---|
| Expense Agent (OCR + Manual) | Add project selection step after expense confirmation | Medium — prompt change could affect vendor matching or account selection intent classification |
| Invoice Agent (both nodes) | Add project selection step after invoice submission | Medium — must not interfere with delivery prompt timing |
| Reporting Agent | Add project P&L intent recognition | Low — additive only, does not change existing report generation paths |

**Mitigation:** Run the full existing intent classification test suite against updated prompts before merging. Use Claude's evaluation tooling if available. Do not ship prompt changes without regression sign-off.

---

## Web App — Technical Additions

```
src/features/projects/
  ├── components/
  │   ├── ProjectsTable.tsx
  │   ├── ProjectDetailPanel.tsx
  │   ├── ProjectPnlChart.tsx
  │   └── ProjectFilterDropdown.tsx
  ├── hooks/
  ├── screens/
  │   └── ProjectsScreen.tsx
  └── services/
      └── _api/
          └── queries.tsx   (useGetProjectsQuery, useGetProjectPnlQuery)

src/app/(dashboard)/dashboard/projects/
  └── page.tsx
```

SDK update required: `@alpha.apps/pinto-web-app-sdk` must be regenerated after backend endpoints are deployed. Frontend work can begin in parallel using a local mock until the SDK is ready.

---

## Configurable Project Limit

`max_projects_per_org` should be stored in the organisations table or a config table (not hardcoded). Default value: `[NEEDS INPUT — product decision]`. Backend enforces this limit at `POST /projects`. The limit should be overridable per org (for future pricing tiers).

---

## Performance Considerations

| Concern | Assessment |
|---|---|
| `list_projects` called on every expense/invoice flow | Low risk — query is trivially fast (most orgs will have <20 projects). Add Redis cache with short TTL (e.g., 60s) if call frequency becomes a concern at scale. |
| Project P&L aggregation on large orgs | Medium risk for orgs with thousands of line items. Index on `project_id` mitigates this. Consider pre-aggregating nightly if response times degrade. |
| Wafeq cost center creation on project create | Adds ~200–400ms to project creation flow. Acceptable for a one-time setup action. |

---

## Deployment Sequence

1. **DB migration** — add `projects` table, add nullable `project_id` columns to `bills` and `invoices`
2. **Backend API** — deploy new project endpoints and P&L aggregation
3. **Wafeq integration** — deploy cost center create/update/tag logic
4. **Zoho integration** — deploy after Zoho sandbox investigation is complete
5. **n8n tool workflows** — deploy 5 new tool workflows
6. **Agent prompt updates** — deploy after regression testing
7. **Web App SDK regeneration** — after backend deploy
8. **Web App** — deploy new Projects route and components
9. **QA** — full end-to-end across Wafeq orgs, Zoho orgs, WhatsApp, and Web App

Steps 1–2 can ship independently (no user-facing impact). Steps 3–6 must ship together to avoid broken agent flows. Steps 7–9 can follow independently.

---

## Open Items Requiring CTO Decision

| # | Item | Options |
|---|---|---|
| 1 | Zoho project creation model — standalone entity or Reporting Tags? | Test in Zoho sandbox; decide before sprint start |
| 2 | Default `max_projects_per_org` limit | Product decision — suggest starting at 20 |
| 3 | P&L aggregation: live query vs. nightly pre-aggregation | Start with live query; add pre-aggregation if performance degrades |
| 4 | Retroactive re-tagging — v1 or v2? | Recommend v2; adds complexity and requires accountant dashboard changes |
| 5 | Project name uniqueness — per org or global? | Per org (already reflected in DB unique index above) |
