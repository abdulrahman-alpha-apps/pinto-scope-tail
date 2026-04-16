---
title: "9. Connections to Existing Tails"
publish: true
---

# Connections to Existing Tails

## Tails This Feature Directly Touches

### 1. Expense Agent Tail
**Connection type:** Modification
**What changes:** Two new tools added (`list_projects`, `assign_project`). Agent prompt updated to introduce optional project selection step after expense confirmation. Both OCR Agent and Manual Input Agent sub-nodes are affected.
**Tail update needed:** Yes — update the Expense Agent tail to document the new tool set, the updated tool call sequence, and the new prompt behaviour around project selection.

---

### 2. Invoice Agent Tail
**Connection type:** Modification
**What changes:** Two new tools added (`list_projects`, `assign_project`). Agent prompt updated to introduce optional project selection step after invoice submission and before delivery. Both invoice agent AI nodes (direct-entry and router-triggered) are affected.
**Tail update needed:** Yes — update the Invoice Agent tail to document the new tools and the updated post-submission flow.

---

### 3. Reporting Agent Tail
**Connection type:** Modification
**What changes:** Two new tools added (`list_projects`, `generate_project_pnl_report`). Agent prompt updated to recognise project P&L intent patterns. New reporting path added for project-level summaries and PDF generation.
**Tail update needed:** Yes — update the Reporting Agent tail to document the new project P&L intent, the new tool, and the expected output format.

---

### 4. Router Agent Tail
**Connection type:** Informational (no structural change in v1)
**What changes:** None in v1. Project creation and management intents are handled inside the Expense/Invoice agents or as a direct user message. If a standalone project management intent branch is added in v1.1, the Router tail will need updating.
**Tail update needed:** Minor note only — flag that project-related intent is handled downstream in v1.

---

### 5. Backend Tail
**Connection type:** Extension
**What changes:** New `projects` entity, new CRUD API endpoints, new `project_id` FK on bills and invoices, Wafeq cost center integration module, Zoho project integration, P&L aggregation logic.
**Tail update needed:** Yes — update the Backend tail to document the new data model, new endpoints, and the Wafeq/Zoho integration pattern for cost centers and project IDs.

---

### 6. Frontend / Web App Tail
**Connection type:** Extension
**What changes:** New `/dashboard/projects` route, new `Projects` sidebar nav item, new `ProjectsTable`, `ProjectDetailPanel`, `ProjectPnlChart`, `ProjectFilterDropdown` components. New React Query hooks for project endpoints. Project filter added to invoice and bill list pages.
**Tail update needed:** Yes — update the Frontend tail to document the new route, new feature module structure under `src/features/projects/`, and the new API hooks.

---

### 7. Segmentation / Target Segments Tail
**Connection type:** Informational
**What changes:** This feature directly improves Pinto's coverage score for:
- Digital Agencies (80% → higher)
- Construction Subcontractors (65% → higher)
- Professional Services (80% → higher)
- Auto Services (70% → higher)
- Transport & Delivery (65% → higher)
**Tail update needed:** Yes — update coverage percentages and remove "Project/job tagging" from the critical blockers column for affected segments once shipped.

---

### 8. Pinto Features Prioritized Tail
**Connection type:** Status update
**What changes:** "Project/job tagging (Cost center tracking)" moves from backlog to in-progress / shipped.
**Tail update needed:** Yes — update the feature status from backlog to in-progress when development begins, and to shipped when released.

---

## New Tails to Create After This Feature Ships

| Tail | Purpose |
|---|---|
| **Cost Center / Projects Tail** | Document the project data model, the WhatsApp flows (create, tag, report), the Wafeq cost center integration, the Zoho project integration, and the P&L aggregation logic. This becomes the reference tail for any future project-related features (billable expense tracking, project budgeting, etc.). |

---

## Downstream Features This Tail Enables

Once Cost Center / Project Tagging is live, the following backlog items become significantly easier to scope and build:

| Feature | How Cost Centers Enable It |
|---|---|
| **Billable vs. non-billable expense tracking** (Professional Services gap) | Cost center is the foundation — tag expense to project, then flag as billable or non-billable |
| **Custom accounting queries** | "Show me all expenses for Project X" becomes a natural query once project_id is on all records |
| **Corporate tax-ready reporting** | Project-level P&L feeds into segment-level and company-level tax profit calculation |
| **Dashboard redesign with exports** | Project P&L view is a high-value export target — PDF/Excel export per project |
| **New standard report types** | Project profitability report is a new standard report type that this feature enables |
