---
title: "7. Function Scoping.e - Testing"
publish: true
---

# Function Scoping — e. Testing

## Functional Goal / Objective
Validate that project creation, tagging, aggregation, and reporting work correctly end-to-end across both accounting providers (Wafeq and Zoho), both agent flows (expense and invoice), and all user-facing surfaces (WhatsApp and Web App). Ensure that optional tagging never blocks or degrades the existing expense/invoice flows.

---

## User Benefit Summary
Incorrect project tagging is silent data corruption — a user believes a transaction is tracked against a project, but it isn't. Testing must be thorough enough that tagged data is trustworthy before the feature ships to project-based segments (agencies, construction, professional services) who will rely on it for business decisions.

---

## Key Test Cases

### A. Project Management

| # | Test Case | Expected Result |
|---|---|---|
| A1 | User creates a project with a valid name | Project created in Pinto DB, Wafeq cost center created, `wafeq_cost_center_id` stored |
| A2 | User creates a project with a name > 200 characters | Backend rejects with validation error; agent informs user to use a shorter name |
| A3 | User creates a duplicate project name | Agent detects duplicate, asks for clarification — does not create a second entry |
| A4 | User creates a project at the org's project limit | Backend rejects; agent informs user and suggests closing an existing project |
| A5 | User closes a project | Status set to `closed` in Pinto DB; Wafeq cost center `is_active` set to `false`; project no longer appears in active list |
| A6 | User lists projects when none exist | Agent skips project question in all flows; no error |

---

### B. Expense Tagging — WhatsApp

| # | Test Case | Expected Result |
|---|---|---|
| B1 | OCR expense flow — user selects project by number | `project_id` stored on expense record; Wafeq bill line item includes `cost_center` field |
| B2 | OCR expense flow — user types project name (partial match) | Agent matches correctly and confirms before tagging |
| B3 | OCR expense flow — user says "skip" / "general" | Expense tagged to General; no `cost_center` on Wafeq line item |
| B4 | Manual expense flow — user selects project | Same as B1 |
| B5 | Expense flow — user has no projects | Project selection question is not asked; flow completes normally |
| B6 | `list_projects` backend call fails (timeout/error) | Agent proceeds without project question; expense tagged to General; no error shown to user |
| B7 | User names a project that does not exist during tagging | Agent asks "Did you mean [closest match]?" with create option |

---

### C. Invoice Tagging — WhatsApp

| # | Test Case | Expected Result |
|---|---|---|
| C1 | Invoice created — user selects project by number | `project_id` stored; Wafeq invoice line items include `cost_center`; Zoho invoice line items include `project_id` |
| C2 | Invoice created — user selects "General" | No project tag on line items |
| C3 | Invoice created — user has no projects | Project question skipped; delivery prompt appears immediately |
| C4 | Invoice with multiple line items — project tagged | All line items on the invoice carry the same cost center / project ID |
| C5 | Invoice tagging — Wafeq org | `cost_center` field present on all line items in Wafeq payload |
| C6 | Invoice tagging — Zoho org | `project_id` field present on all line items in Zoho payload |

---

### D. Project P&L Reporting — WhatsApp

| # | Test Case | Expected Result |
|---|---|---|
| D1 | User requests P&L for a specific project by name | Reporting agent returns revenue, costs, net profit, margin |
| D2 | User requests P&L with date range ("this month", "Q1") | Aggregation filters correctly by date; numbers match expected values |
| D3 | Project with revenue only (no expenses tagged) | Costs = 0, Net Profit = Revenue, no division-by-zero error |
| D4 | Project with costs only (no invoices tagged) | Revenue = 0, Net Profit = negative cost value |
| D5 | Project with no transactions | Returns empty state message: "No transactions tagged to [project] yet." |
| D6 | User requests PDF report for a project | PDF generated and delivered via WhatsApp; PDF contains correct project data |
| D7 | "General" project P&L requested | Returns aggregation of all untagged transactions |
| D8 | User asks for overall project comparison ("which project is most profitable?") | Agent lists top projects by net profit |

---

### E. Web App Dashboard

| # | Test Case | Expected Result |
|---|---|---|
| E1 | Projects page loads with active projects | Table shows correct revenue, cost, profit, margin per project |
| E2 | Project detail drawer opens | Invoice and expense breakdowns load correctly; totals match P&L summary |
| E3 | Date filter applied on projects page | All figures update correctly for the selected period |
| E4 | Projects page with no projects | Empty state message displayed; no errors |
| E5 | Closed project in filter | Appears only when "All" or "Closed" filter is selected |
| E6 | Dashboard overview card | Top 3 projects by margin shown correctly |
| E7 | Invoice list page — project filter | Filters invoices to only those tagged to the selected project |
| E8 | Bills/expenses list page — project filter | Same as E7 for expenses |

---

### F. Accountant Dashboard

| # | Test Case | Expected Result |
|---|---|---|
| F1 | Accountant reviews a bill task — project is tagged | Project name visible in the bill form (read-only, informational) |
| F2 | Accountant resolves a bill — project tag preserved | Wafeq bill posted with correct `cost_center` on line items |
| F3 | Task created for a bill that has no project | Project field shows "General" or is empty — no error |

---

## Edge Cases

| Scenario | Expected Behaviour |
|---|---|
| User tags expense to Project A, then the project is closed before the expense is posted | Expense retains the project_id; post proceeds; historical data preserved |
| Multi-currency invoice tagged to a project | P&L aggregation converts to AED using stored exchange rate at transaction time |
| Same vendor appears in multiple projects | Vendor matching is unaffected; project tag is independent of vendor |
| Wafeq cost center creation fails during project creation | Pinto rolls back local project record; agent informs user of failure; retries should be possible |
| User edits a Wafeq bill directly in Wafeq (not via Pinto) and removes the cost center | Pinto's DB still shows the project tag; sync discrepancy — [NEEDS INPUT] on reconciliation strategy |
| Organisation switches accounting provider mid-flight (Wafeq → Zoho) | New transactions use Zoho `project_id`; historical Wafeq records retain `wafeq_cost_center_id` |

---

## Integration Test Requirements

| Integration | Test Environment | What to Verify |
|---|---|---|
| Wafeq | Wafeq sandbox / test org | Cost center CRUD; bill line item `cost_center` field accepted; invoice line item field accepted |
| Zoho Books | Zoho sandbox | `project_id` field accepted on invoice line items; project creation API response |
| WATI | WATI test environment | Project selection numbered list renders correctly on WhatsApp; user input correctly parsed |
| n8n | Dev environment | All 3 modified agents complete flows without errors; `list_projects` returns correctly; `assign_project` sets correct DB state |

---

## Regression Risks

| Risk | Affected Flow |
|---|---|
| `list_projects` call slows down expense/invoice flows | OCR expense flow, manual expense flow, invoice flow — measure latency before/after |
| Agent prompt change causes misclassification of existing intents | All agent flows — run existing intent classification tests after prompt updates |
| `cost_center` field on Wafeq line items breaks existing bill submission | All expense recording flows — must test Wafeq payload compatibility with and without cost center |
| New `project_id` DB column causes migration issues on existing records | All existing expense and invoice records — ensure nullable FK does not break existing queries |
| Project filter on invoice/bill list pages breaks existing filter logic | Invoice list, bill list pages in Web App |
