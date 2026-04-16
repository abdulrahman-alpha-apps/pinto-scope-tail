---
title: "2. User Interaction"
publish: true
---

# User Interaction — Cost Center / Project-Level P&L Tracking

## Interface
**WhatsApp only** (via WATI → n8n Agent).
The Web App dashboard displays project P&L as a read-only reporting view — no project management actions are taken from the Web App.

---

## Flow 1 — Creating a Project

**Trigger:** User sends a message expressing intent to create a new project.

**Example triggers:**
- "Create a new project called Alpha Fitout"
- "Add a job: Villa 34 Renovation"
- "I want to track a new client project"

**Conversation flow:**
```
User:    "Create a project called Alpha Fitout"
Pinto:   "Got it! Project 'Alpha Fitout' has been created.
          You can now tag your expenses and invoices to this project."
```

If the name is ambiguous or too short, Pinto asks for confirmation before creating.

---

## Flow 2 — Tagging an Expense to a Project

**Trigger:** User uploads a receipt or manually enters an expense. After vendor and account are confirmed, Pinto optionally asks for project assignment.

**Conversation flow (OCR path):**
```
User:    [uploads receipt photo]
Pinto:   "Got it — AED 3,200 from Gulf Cement Supplies.
          Expense category: Raw Materials.
          Which project should I tag this to?
          1. Alpha Fitout
          2. Marina Tower Job
          3. General (no project)
          Reply with the number or project name."
User:    "1"
Pinto:   "Done! Expense tagged to Alpha Fitout and recorded."
```

**Conversation flow (manual path):**
Same flow — project selection is the final step after category and vendor are confirmed.

**Rules:**
- Project selection is optional. If the user skips or says "general", the expense is tagged to "General".
- If the user has no projects created yet, Pinto skips the project question entirely.
- If the user has one project only, Pinto asks "Tag to [Project Name] or General?"

---

## Flow 3 — Tagging an Invoice to a Project

**Trigger:** User creates an invoice. After customer, items, and amounts are confirmed, Pinto optionally asks for project assignment.

**Conversation flow:**
```
Pinto:   "Invoice created for Falcon Properties — AED 45,000.
          Which project should I tag this to?
          1. Alpha Fitout
          2. Marina Tower Job
          3. General (no project)"
User:    "Alpha Fitout"
Pinto:   "Done! Invoice tagged to Alpha Fitout. Should I send it to the customer now?"
```

---

## Flow 4 — Viewing Project P&L

**Trigger:** User requests a report for a specific project.

**Example triggers:**
- "Show me the P&L for Alpha Fitout"
- "How profitable is the Marina Tower job?"
- "What's my profit on Project X this month?"

**Conversation flow:**
```
User:    "Show me the P&L for Alpha Fitout"
Pinto:   "Here's the summary for Alpha Fitout:

          Revenue:    AED 145,000
          Costs:      AED 98,400
          Net Profit: AED 46,600 (32% margin)

          Period: All time
          Want me to filter by a specific date range, or send a full PDF report?"
User:    "Send the PDF"
Pinto:   [delivers PDF report via WhatsApp]
```

---

## Flow 5 — Listing All Projects

**Trigger:** User asks to see their projects.

**Example triggers:**
- "Show me my projects"
- "List all jobs"
- "What projects do I have?"

**Conversation flow:**
```
User:    "Show me my projects"
Pinto:   "You have 3 active projects:
          1. Alpha Fitout
          2. Marina Tower Job
          3. Client ABC Retainer

          Reply with a project name to see its P&L, or say 'new project' to create one."
```

---

## Flow 6 — Renaming or Closing a Project

**Trigger:** User wants to rename or mark a project as complete/inactive.

```
User:    "Close the Alpha Fitout project"
Pinto:   "Alpha Fitout has been marked as closed. It will no longer appear in active project lists,
          but all historical data is preserved and can still be reported on."
```

---

## Edge Cases

| Scenario | Behaviour |
|---|---|
| User has no projects | Project selection question is skipped entirely |
| User types a project name not in the list | Pinto asks "Did you mean [closest match]?" with confirm/create options |
| User says "skip" or "no project" | Tagged to General |
| Project limit reached | Pinto informs user of the limit and offers to close an existing project first |
| Untagged historical transactions | Remain under "General"; cannot be retroactively tagged via WhatsApp in v1 |
