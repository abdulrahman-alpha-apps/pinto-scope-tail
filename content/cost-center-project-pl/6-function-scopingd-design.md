---
title: "6. Function Scoping.d - Design"
publish: true
---

# Function Scoping — d. Design

## Functional Goal / Objective
Design the end-to-end user experience for project creation, project tagging during expense and invoice flows, and project P&L reporting — across WhatsApp and the Web App dashboard. Design must feel like a natural, lightweight addition to existing Pinto flows, not a new product module bolted on.

---

## User Benefit Summary
Good design here means that project tagging adds minimal friction to the existing expense and invoice flows — one extra question, clearly worded, with a numbered list the user can answer in one tap. The Web App view makes profitability visible at a glance without requiring the user to ask. If either of these feels heavy or confusing, adoption will be low and tagging will be skipped.

---

## WhatsApp Chat Flow Design

### Principle
Project selection is always the **last step** of an existing flow, never an interruption in the middle. It appears after the core transaction is confirmed. It is skippable in one word ("skip", "general", "no project").

---

### Chat Flow 1 — Expense with Project Tag

```
[After vendor and category are confirmed]

Pinto: "AED 3,200 from Gulf Cement Supplies — Raw Materials.
        Which project should I tag this to?

        1. Alpha Fitout
        2. Marina Tower Job
        3. General (no project)

        Reply with a number or project name."

User:  "1"

Pinto: "Done ✓ Expense tagged to Alpha Fitout and recorded."
```

**Design notes:**
- Numbered list — clean, mobile-friendly, one keystroke answer
- "General" is always the last option and visually labelled as the skip path
- Confirmation message is short — user knows it's done in one line
- If user has no projects: skip the question entirely, proceed to the standard confirmation

---

### Chat Flow 2 — Invoice with Project Tag

```
[After invoice is created and before delivery prompt]

Pinto: "Invoice created — AED 45,000 for Falcon Properties.
        Which project is this for?

        1. Alpha Fitout
        2. Marina Tower Job
        3. General (no project)"

User:  "Alpha Fitout"

Pinto: "Tagged to Alpha Fitout.
        Should I send this invoice to Falcon Properties now?"
```

**Design notes:**
- Project name match is fuzzy — agent accepts "Alpha" as a match for "Alpha Fitout"
- The delivery prompt follows immediately after tagging — the flow feels unbroken

---

### Chat Flow 3 — Create a Project

```
User:  "Create a new project called Villa 34 Renovation"

Pinto: "Project 'Villa 34 Renovation' created.
        You can now tag expenses and invoices to it."
```

Short. No unnecessary confirmation steps.

---

### Chat Flow 4 — Project P&L Report

```
User:  "Show me profit for Alpha Fitout"

Pinto: "Alpha Fitout — All time

        Revenue:      AED 145,000
        Costs:         AED 98,400
        ─────────────────────────
        Net Profit:    AED 46,600
        Margin:             32%

        Want a PDF report, or filter by date range?"
```

**Design notes:**
- Plain text table — readable on mobile without formatting issues
- Horizontal rule separates revenue/cost from the net line — makes the profit figure visually distinct
- Clear follow-up options offered in the same message
- Negative profit: clearly labelled "Net Loss: AED -4,200" in red-adjacent language (no emoji colours — keep neutral)

---

### Chat Flow 5 — List Projects

```
User:  "Show me my projects"

Pinto: "Your active projects:

        1. Alpha Fitout
        2. Marina Tower Job
        3. Villa 34 Renovation

        Reply with a name to see its P&L, or say 'new project' to create one."
```

---

## Web App Design

### Projects Page (`/dashboard/projects`)

**Layout:** Standard dashboard page layout — same header, sidebar, and container width as existing pages.

**Table design:**

| Column | Format | Notes |
|---|---|---|
| Project Name | Text | Clickable — opens detail panel |
| Revenue | AED formatted | Right-aligned |
| Costs | AED formatted | Right-aligned |
| Net Profit | AED formatted | Colour-coded: green if positive, red if negative |
| Margin | % | Colour-coded: green >20%, amber 5–20%, red <5% |
| Status | Chip: Active / Closed | Subtle, not dominant |

**No actions in the table** (no edit/delete buttons) — project management is WhatsApp-only in v1. A subtle helper text below the table: "To create or rename projects, message Pinto on WhatsApp."

---

### Project Detail Panel

Opens as a **right-side drawer** (consistent with existing document preview patterns in the codebase).

**Panel sections:**

1. **Header** — Project name, status chip, total margin chip
2. **P&L Summary** — Revenue / Costs / Net Profit as three stat cards (reuse existing dashboard stat card component pattern)
3. **P&L Chart** — Bar chart: Revenue vs Costs by month (recharts, reuse existing chart style)
4. **Invoices tab** — Table of invoices tagged to this project: invoice number, customer, amount, status, date
5. **Expenses tab** — Table of expenses tagged to this project: vendor, category, amount, date

**Date filter** at the top of the panel applies to all sections.

---

### Dashboard Overview Card

A new "Top Projects" card on the main `/dashboard` overview:

```
┌─────────────────────────────────────┐
│ Top Projects         This month  ▼  │
├─────────────────────────────────────┤
│ Alpha Fitout            +32% margin │
│ Marina Tower Job         +5% margin │
│ Villa 34 Renovation    -12% margin  │
├─────────────────────────────────────┤
│              View all projects →    │
└─────────────────────────────────────┘
```

---

## Component Reuse

| Component to Reuse | Where |
|---|---|
| Existing stat cards (dashboard summary) | Project detail P&L summary |
| Existing recharts line/bar chart | Project P&L monthly chart |
| Existing document preview drawer | Project detail drawer pattern |
| Existing table with filters | Projects list table |
| Existing filter chips (date range, status) | Projects page filters |
| Existing MUI `Chip` component | Project status indicator |

---

## Interaction Patterns

- **Click project row** → opens detail drawer (no page navigation — keeps context)
- **Date filter change** → refetches data, no page reload
- **Tab switch (Invoices / Expenses)** → instant, no network call (data already loaded in drawer)
- **"View all projects" link** on dashboard card → navigates to `/dashboard/projects`
- **Empty state** (no projects yet): friendly message — "No projects yet. Message Pinto on WhatsApp to create your first project."

---

## Constraints

- No create/edit/delete UI in the Web App in v1.
- Arabic project names must render correctly (RTL support already in place via `stylis-plugin-rtl`).
- Colour coding for margin health must use existing MUI palette tokens only (`success.main`, `warning.main`, `error.main`).
- Mobile: projects table collapses to card list — show project name and margin only. Tap to open detail drawer (full screen on mobile).
