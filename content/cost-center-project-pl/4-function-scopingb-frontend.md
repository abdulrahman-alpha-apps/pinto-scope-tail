---
title: "4. Function Scoping.b - Frontend"
publish: true
---

# Function Scoping — b. Frontend (Web App)

## Functional Goal / Objective
The Web App must surface project-level P&L data as a read-only reporting view inside the existing dashboard. Users should be able to see all their projects, select a project, and view its revenue, costs, and net profit — with optional date filtering. No project management actions (create, rename, close) are taken from the Web App in v1; those happen via WhatsApp.

---

## User Benefit Summary
Business owners who open the Web App can immediately see which projects are profitable and which are not, without needing to start a WhatsApp conversation. This is the "always-on visibility" complement to the WhatsApp reporting flow — the same data, available at a glance on the dashboard.

---

## Screens Affected

### 1. Dashboard Overview Page (`/dashboard`)
**Change:** Add a "Projects" summary card to the existing dashboard overview.

**Card contents:**
- Top 3 projects by net profit (current month)
- Each row: project name, revenue, cost, net profit, margin %
- "View all projects" link → routes to `/dashboard/projects`

---

### 2. New Page — Projects P&L (`/dashboard/projects`)

**New route** inside the `(dashboard)` route group. Add to the sidebar navigation as "Projects".

**Page layout:**

```
┌─────────────────────────────────────────────────┐
│  Projects                          [Date filter] │
├─────────────────────────────────────────────────┤
│  Project Name     Revenue    Costs   Net   Margin│
│  ─────────────────────────────────────────────  │
│  Alpha Fitout     145,000   98,400  46,600  32%  │
│  Marina Tower      72,000   68,200   3,800   5%  │
│  General           18,500   12,100   6,400  35%  │
├─────────────────────────────────────────────────┤
│  [Click any row → Project Detail view]           │
└─────────────────────────────────────────────────┘
```

**Filters:**
- Date range (from / to) — using existing `@mui/x-date-pickers` components
- Project status: Active / Closed / All

---

### 3. New Component — Project Detail Panel

When a user clicks a project row, a detail panel (drawer or expanded row) opens showing:

- Revenue breakdown: list of invoices tagged to this project (invoice number, customer, amount, date, status)
- Cost breakdown: list of expenses/bills tagged to this project (vendor, category, amount, date)
- Net P&L chart (recharts bar or line chart — reuse existing chart components from dashboard)
- Date range is inherited from the page-level filter

---

### 4. Existing Views — Project Filter Addition

Add an optional "Project" filter chip/dropdown to:

| View | Filter Added |
|---|---|
| `/dashboard/pending-invoices` | Project filter dropdown |
| `/dashboard/pending-bills` | Project filter dropdown |

This allows users to drill into open items for a specific project directly from existing views.

---

## New Components

| Component | Description |
|---|---|
| `ProjectsTable` | Table with project rows, sortable by revenue / cost / profit / margin |
| `ProjectDetailPanel` | Drawer/expanded panel showing invoice and expense breakdown per project |
| `ProjectPnlChart` | Bar chart (revenue vs costs) per project — built on existing `recharts` setup |
| `ProjectFilterDropdown` | Reusable project selector chip for invoice/bill list filters |

All components follow the existing Feature-Sliced Design pattern under `src/features/projects/`.

---

## API Hooks Required (React Query)

| Hook | Endpoint | Description |
|---|---|---|
| `useGetProjectsQuery` | `GET /projects` | List all projects for the org |
| `useGetProjectPnlQuery` | `GET /projects/:id/pnl` | P&L summary for one project |
| `useGetAllProjectsPnlQuery` | `GET /projects/pnl` | Aggregated P&L for all projects (dashboard card) |

Follow existing `createQuery` factory pattern in `src/libs/tanstack-helpers.ts`.

---

## Vibe-Coding Brief

**Design direction:** Clean data-dense table with a subtle colour signal for margin health.
- Margin > 20%: green accent
- Margin 5–20%: amber accent
- Margin < 5% or negative: red accent
- Consistent with existing MUI theme palette — do not introduce new colours, use existing `success`, `warning`, `error` palette tokens.
- No new icon libraries. Use existing `@mui/icons-material`.
- The Projects page should feel like a natural extension of the existing Reports page — same layout shell, same table patterns.
- Mobile: table collapses to card list (revenue and margin visible, costs hidden behind expand). Follow existing responsive patterns in the codebase.

---

## Responsive Considerations
- The Web App already has RTL support via `stylis-plugin-rtl`. The project name must render correctly in both LTR (English) and RTL (Arabic) if Arabic project names are used (Wafeq supports `name_ar`).
- Date pickers: reuse `@mui/x-date-pickers` already in the project.
- No new chart libraries — extend existing `recharts` usage.

---

## Constraints

- In v1, project creation/rename/close is **WhatsApp only** — no create/edit forms in the Web App.
- The Web App is a **static export** (Next.js 14, `output: 'export'`). All data must come from API calls, not server-side rendering.
- `tasks-management` feature is currently excluded from the build (WIP) — the new Projects feature must be added independently and not depend on it.

---

## Connections & Integrations

| Connects To | How |
|---|---|
| Backend `GET /projects` | Fetch project list |
| Backend `GET /projects/:id/pnl` | Fetch per-project P&L |
| Existing dashboard layout | New sidebar nav item, new route group entry |
| Existing invoice/bill list views | Add project filter dropdown to existing filter bars |
| Existing `recharts` charts | Reuse for project P&L bar chart |
