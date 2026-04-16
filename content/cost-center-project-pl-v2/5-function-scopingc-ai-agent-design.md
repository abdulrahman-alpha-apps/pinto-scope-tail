---
title: "5. Function Scoping.c - AI Agent Design"
publish: true
---

# Function Scoping — c. AI Agent Design

## Functional Goal / Objective
Three existing specialist agents must be modified to support cost center / project tagging: the **Expense Agent**, the **Invoice Agent**, and the **Reporting Agent**. A new set of shared tools — `list_projects`, `create_project`, `assign_project` — must be built and connected to the relevant agents. No new top-level agent is needed. The Router Agent requires no structural changes.

---

## User Benefit Summary
The agents are the user's only interface for project management and tagging (WhatsApp is the primary and only surface for v1). Without agent changes, users have no way to create projects, tag transactions, or request project-level reports through conversation. The agent modifications are what make this feature real for users.

---

## New Tools Required

### Tool 1 — `list_projects`
**Used by:** Expense Agent, Invoice Agent, Reporting Agent

**Purpose:** Retrieve the list of active projects for the current organisation, to present as options during tagging or reporting flows.

**Behaviour:**
- Calls `GET /projects?org_id=...&status=active` on the Pinto backend
- Returns: array of `{ id, name }` objects
- If empty: returns empty array — agent skips the project question entirely
- Agent formats the list as a numbered WhatsApp message for the user to select from

---

### Tool 2 — `create_project`
**Used by:** Router Agent (via a new intent branch) or directly by Expense/Invoice Agent if user expresses intent mid-flow

**Purpose:** Create a new named project for the organisation.

**Behaviour:**
- Accepts: `{ name: string }`
- Validates: name ≤ 200 characters, not a duplicate of an existing project name
- Calls `POST /projects` on the Pinto backend
- Backend simultaneously creates the Wafeq cost center and (if Zoho org) the Zoho project
- Returns: `{ id, name, status }` of the created project
- Agent confirms: "Project '[name]' has been created."

**Error cases:**
- Duplicate name → Agent asks: "You already have a project called '[name]'. Did you mean that one, or would you like a different name?"
- Project limit reached → Agent informs user and suggests closing an inactive project first

---

### Tool 3 — `assign_project`
**Used by:** Expense Agent (after expense is confirmed), Invoice Agent (after invoice is created)

**Purpose:** Set the `project_id` on a draft expense or invoice before the final record is posted to Wafeq/Zoho.

**Behaviour:**
- Accepts: `{ record_type: 'expense' | 'invoice', record_id: string, project_id: string | 'general' }`
- Stores `project_id` on the pending record in Pinto DB
- When the record is subsequently posted to Wafeq/Zoho, the backend includes the cost center / project tag on the line items
- Returns: confirmation

---

### Tool 4 — `close_project` (rename to inactive)
**Used by:** Dedicated project management intent (Router → dispatched directly)

**Purpose:** Mark a project as closed/inactive.

**Behaviour:**
- Accepts: `{ project_id: string }`
- Calls `PATCH /projects/:id` with `{ status: 'closed' }`
- Calls Wafeq `PATCH /cost_centers/{id}` with `{ is_active: false }`
- Returns confirmation

---

## Agent Modifications

### Expense Agent
**File:** `Expense agent V3.json`

**Changes required:**
1. Add `list_projects` tool to both the **OCR Agent** and the **Manual Input Agent** sub-nodes.
2. Add `assign_project` tool to both sub-nodes.
3. Update agent prompt: after expense account and vendor are confirmed, the agent should call `list_projects`. If the result is non-empty, present options and call `assign_project` with the user's selection before calling `record_expense`. If the user skips, call `assign_project` with `project_id: 'general'`.

**Tool call sequence (OCR path):**
```
OCR → confirm vendor → confirm account → list_projects
  → [if projects exist] present options → user selects
  → assign_project → record_expense → send confirmation
```

**Tool call sequence (manual path):**
```
collect fields → confirm → list_projects
  → [if projects exist] present options → user selects
  → assign_project → record_expense → send confirmation
```

---

### Invoice Agent
**File:** `Invoice agent.json`

**Changes required:**
1. Add `list_projects` tool to both Invoice Agent AI nodes (the direct-entry and router-triggered paths).
2. Add `assign_project` tool to both AI nodes.
3. Update agent prompt: after `Submit invoice` is called (invoice created), present project options. Call `assign_project` before calling `Invoice Delivery Tool` (so the project tag is in place before delivery confirmation).

**Tool call sequence:**
```
match customer → match items → select account → submit_invoice
  → list_projects → [if projects exist] present options → user selects
  → assign_project → invoice_delivery_tool → send confirmation
```

---

### Reporting Agent
**File:** `Reporting agent.json`

**Changes required:**
1. Add `list_projects` tool to the Reporting Agent AI node.
2. Add a new tool: `generate_project_pnl_report` (calls `GET /projects/:id/pnl` on the backend, formats the result, and optionally triggers PDF generation).
3. Update agent prompt: recognise project P&L intents ("Show me profit for [project]", "How is [project] doing?") and route to `generate_project_pnl_report`.

**New tool: `generate_project_pnl_report`**
- Accepts: `{ project_id: string, date_from?: string, date_to?: string }`
- Calls `GET /projects/:id/pnl` with optional date params
- Returns: structured P&L summary (revenue, costs, net profit, margin %)
- Agent formats as WhatsApp message or triggers PDF delivery via existing report delivery flow

---

### Router Agent
**No structural changes required.**

The Router's existing intent classification already handles the relevant domains:
- Expense-related project creation → stays in Expense Agent or is a standalone "project management" intent
- Invoice-related project creation → Invoice Agent
- Reporting → Reporting Agent

However, if "create project" becomes a frequent standalone trigger (no concurrent expense/invoice flow), consider adding a lightweight **project management intent branch** in the Router that calls `create_project` directly without entering a full specialist agent. This is a **v1.1 consideration**, not required for v1.

---

## n8n Workflow Changes

| Workflow | Change Type | Description |
|---|---|---|
| Expense agent V3 | Modification | Add `list_projects` and `assign_project` tool nodes; update AI Agent prompt |
| Invoice agent | Modification | Add `list_projects` and `assign_project` tool nodes; update both AI Agent nodes |
| Reporting agent | Modification | Add `list_projects` and `generate_project_pnl_report` tool nodes; update AI Agent prompt |
| New: `list_projects` tool workflow | New tool workflow | HTTP call to `GET /projects` — shared across agents |
| New: `create_project` tool workflow | New tool workflow | HTTP call to `POST /projects` |
| New: `assign_project` tool workflow | New tool workflow | HTTP call to `PATCH /expenses/:id` or `PATCH /invoices/:id` with project_id |
| New: `generate_project_pnl_report` tool workflow | New tool workflow | HTTP call to `GET /projects/:id/pnl`, formats output |
| New: `close_project` tool workflow | New tool workflow | HTTP call to `PATCH /projects/:id` |

---

## MCP Connectors / Skills Required
No new MCP connectors are required. All new tools call the Pinto backend via HTTP (same pattern as existing tool workflows).

---

## Agent Routing Changes
- No changes to the Router's top-level dispatch logic in v1.
- Project selection is handled **inline** within the Expense and Invoice agent flows — it is not a separate routing destination.
- Reporting Agent handles project P&L requests via its existing entry path.

---

## Constraints

- If `list_projects` returns an empty list, agents must **not** ask the project question — silently assign "General" and proceed.
- Project selection must not block or significantly slow down the existing expense/invoice flows. If the backend call for `list_projects` fails, the agent must proceed without project tagging (fail-open, assign "General").
- The agent must handle free-text project name entry as well as numbered list selection — users may type the project name directly rather than a number.
- Memory: the agent should not remember the user's "last used project" across separate conversations in v1 — this is a v2 enhancement (depends on Context memory across conversations feature).

---

## Connections & Integrations

| Connects To | How |
|---|---|
| Pinto Backend | All new tools call backend REST endpoints via HTTP |
| Expense Agent | `list_projects` + `assign_project` added to both OCR and Manual sub-agents |
| Invoice Agent | `list_projects` + `assign_project` added to both invoice agent nodes |
| Reporting Agent | `list_projects` + `generate_project_pnl_report` added |
| Wafeq / Zoho | Via backend — agents never call Wafeq/Zoho directly |
