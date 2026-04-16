---
title: "pinto-features-prioritized"
publish: true
---

# Pinto Features — Prioritized by Strategic Value

## Priority Framework

Features are prioritized based on:
1. **Impact on top-priority segments** (Trading, Digital Agencies, Construction, E-commerce)
2. **Universal coverage gaps** (features that unlock multiple segments simultaneously)
3. **Revenue potential** ($375K+ segment needs)
4. **Strategic dependencies** (features that unblock other features)
5. **Quick wins vs. long-term investments**

---

## Feature Types
- **Enhancement** = Feature exists but needs improvement
- **Need Implement** = Feature doesn't exist yet, needs to be built from scratch

---

## 🔴 CRITICAL PRIORITY — Universal Blockers

These features unlock multiple high-priority segments and appear as gaps across all top segments.

| Priority | Feature                                    | Type           | Unlocks Segments                                                                                                     | Strategic Impact                                                                                                                                                                               | Status in Backlog                                                    |
| -------- | ------------------------------------------ | -------------- | -------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| **P0**   | Bank statement processing                  | Need Implement | Trading (⭐⭐⭐⭐⭐), Digital Agencies (⭐⭐⭐⭐⭐), Construction (⭐⭐⭐⭐⭐), E-commerce (⭐⭐⭐⭐⭐), ALL other segments              | **Universal gap** — appears in 13/14 segments. Enables automated reconciliation of bank transactions to recorded entries. Critical for businesses with high transaction volumes.               | ✅ In current backlog as "Bank statement processing"                  |
| **P0**   | Project/job tagging (Cost center tracking) | Need Implement | Digital Agencies (⭐⭐⭐⭐⭐), Construction (⭐⭐⭐⭐⭐), Auto Services (⭐⭐⭐⭐), Professional Services (⭐⭐⭐⭐), Transport (⭐⭐⭐⭐) | Enables project-level P&L tracking. Critical for agencies, construction, and professional services billing models. Without this, these segments cannot track profitability per client/project. | ✅ In current backlog as "Project/job tagging (Cost center tracking)" |
| **P1**   | Multi-currency partial payment handling    | Enhancement    | Trading (⭐⭐⭐⭐⭐), Digital Agencies (⭐⭐⭐⭐⭐), E-commerce (⭐⭐⭐⭐⭐), Real Estate (⭐⭐⭐⭐)                                    | Trading companies handle USD/EUR invoices from international suppliers. Digital agencies invoice international clients. Current system struggles with partial payments in foreign currencies.  | ✅ In current backlog as "Multi-currency partial payment handling"    |
| **P1**   | Refunds and returns                        | Need Implement | E-commerce (⭐⭐⭐⭐⭐), Retail (⭐⭐), F&B (⭐⭐⭐⭐)                                                                          | E-commerce sellers (60% coverage gap) cannot properly reverse revenue entries when refunds occur. Creates accounting discrepancies and blocks segment adoption.                                | ✅ In current backlog as "Refunds and returns"                        |

**Why these are P0/P1:**
- **Bank reconciliation** is mentioned in **13 of 14 segments** — it's the most universal gap
- **Project/job tagging** unlocks **5 high-priority segments** (Digital Agencies, Construction, Professional Services, Auto Services, Transport)
- **Multi-currency payments** and **refunds** are critical blockers for Trading and E-commerce (both 5-star segments)

---

## 🟠 HIGH PRIORITY — Segment-Specific Critical Features

These features unlock specific high-value segments or significantly improve conversion rates.

| Priority | Feature | Type | Unlocks Segments | Strategic Impact | Status in Backlog |
| --- | --- | --- | --- | --- | --- |
| **P1** | Custom accounting queries | Need Implement | ALL segments (especially Trading, Professional Services, Digital Agencies) | Users need to ask "Show me all unpaid invoices for customer X" or "What did I spend with vendor Y last month?" Without this, they must export to Excel. Reduces churn. | ✅ In current backlog as "Custom accounting queries" |
| **P1** | Query accounting system data | Need Implement | ALL segments | Allows Router Agent to answer questions like "Show me invoice #123" or "What's the total I owe vendor ABC?" Requires querying Wafeq/Zoho directly. Significantly improves user experience and reduces support burden. | ✅ In current backlog as "Query accounting system data" |
| **P2** | Un-earned revenue invoices (Advance payments) | Need Implement | Digital Agencies (⭐⭐⭐⭐⭐), Professional Services (⭐⭐⭐⭐), Construction (⭐⭐⭐⭐⭐) | Agencies and professional services often invoice for retainers/deposits. Construction invoices milestones. Without this, revenue recognition is incorrect. | ✅ In current backlog as "Un-earned revenue invoices" |
| **P2** | Prepaid and accrual expense handling | Need Implement | Trading (⭐⭐⭐⭐⭐), Professional Services (⭐⭐⭐⭐), Real Estate (⭐⭐⭐⭐) | Trading companies pay suppliers upfront, expense over time. Professional services prepay office rent, software licenses. Critical for accurate P&L. | ✅ In current backlog as "Prepaid and accrual expense handling" |
| **P2** | Thread management after invoice creation | Enhancement | ALL segments (especially Freelancers, Agencies, Auto Services) | Current issue: threads close after creating an invoice, preventing users from sending it to customers in the same conversation. Major UX friction point. | ✅ In current backlog as "Thread management after invoice creation" |
| **P2** | Backend messaging with context | Need Implement | ALL segments | Backend agents (scheduled jobs, notifications) send messages that break conversation context. Users get confused when Agent doesn't understand the prior exchange. Critical for conversation coherence. | ✅ In current backlog as "Backend messaging with context" |

**Why these are P1/P2:**
- **Custom queries** dramatically improve user experience and reduce the need for manual Excel exports
- **Un-earned revenue** and **prepaid expenses** are accounting fundamentals missing for professional services and trading
- **Thread management** and **backend messaging** are UX blockers that increase frustration and support burden

---

## 🟡 MEDIUM PRIORITY — Quality of Life & Reliability Improvements

These features improve reliability, reduce task creation, and enhance existing workflows.

| Priority | Feature | Type | Segments Impacted | Strategic Impact | Status in Backlog |
| --- | --- | --- | --- | --- | --- |
| **P2** | OCR validation improvements | Enhancement | ALL segments (especially F&B, Trading, Construction, Retail) | Improves document extraction accuracy and UAE tax field validation. Reduces task escalations to accountants. Higher accuracy = faster processing = better user experience. | ✅ In current backlog as "OCR validation improvements" |
| **P2** | Chart of accounts history matching | Enhancement | ALL segments | Add pattern recognition so Agent remembers which Chart of Accounts category a user selected previously for similar expenses. Reduces repetitive questions and speeds up expense recording. | ✅ In current backlog as "Chart of accounts history matching" |
| **P2** | Task creation fallback improvements (Expense) | Enhancement | ALL segments | Improve logic for when/how Agent creates tasks for accountants when it cannot resolve expenses automatically. Currently creates too many tasks, overwhelming accountants. | ✅ In current backlog as "Task creation fallback improvements (Expense)" |
| **P3** | Manual entry flow improvements | Enhancement | ALL segments | Make manual expense entry conversation clearer and easier to follow. Users currently get confused by the flow. | ✅ In current backlog as "Manual entry flow improvements" |
| **P3** | Remaining balance tracking | Enhancement | ALL segments | Better display and tracking of outstanding balances after partial payments. Currently confusing for users. | ✅ In current backlog as "Remaining balance tracking" |
| **P3** | Mark bills as paid during recording | Enhancement | F&B (⭐⭐⭐⭐), Retail (⭐⭐), Auto Services (⭐⭐⭐⭐) | Allow cash payments to be recorded at the time of expense entry (instead of separate payment step). Simplifies workflow for cash-heavy businesses. | ✅ In current backlog as "Mark bills as paid during recording" |

**Why these are P2/P3:**
- Improve reliability and reduce support burden
- Incremental UX improvements rather than segment blockers
- Can be tackled after critical features

---

## 🟢 LOW PRIORITY — Nice-to-Have & Future Enhancements

These features improve experience but are not blockers for segment adoption.

| Priority | Feature | Type | Segments Impacted | Strategic Impact | Status in Backlog |
| --- | --- | --- | --- | --- | --- |
| **P3** | Report delivery improvements | Need Implement | ALL segments | Improve report PDF format and reduce delivery time. Currently works but could be better. | ✅ In current backlog as "Report delivery improvements" |
| **P3** | Custom reports (Human in the loop) | Need Implement | Professional Services (⭐⭐⭐⭐), Trading (⭐⭐⭐⭐⭐) | Enable users to request custom reports that require accountant review. Low volume, manual process acceptable. | ✅ In current backlog as "Custom reports (Human in the loop)" |
| **P3** | New standard report types | Need Implement | ALL segments | Add aging reports, expense breakdowns by category, etc. Nice additions but not critical for adoption. | ✅ In current backlog as "New standard report types" |
| **P3** | Purchase order reference | Need Implement | Professional Services (⭐⭐⭐⭐), Trading (⭐⭐⭐⭐⭐) | Allow users to add PO number to invoices. Requested by some corporate clients but not a blocker. | ✅ In current backlog as "Purchase order reference" |
| **P4** | Context memory across conversations | Need Implement | ALL segments | Remember last 3 messages so users can reference prior conversations. Nice but not critical. | ✅ In current backlog as "Context memory across conversations" |
| **P4** | Improve agent routing | Enhancement | ALL segments | Remove message shown when switching agents + improve trigger keywords. Minor annoyance, not a blocker. | ✅ In current backlog as "Improve agent routing" |

**Why these are P3/P4:**
- Nice improvements but not adoption blockers
- Can be deferred without impacting segment penetration
- Lower ROI compared to P0-P2 features

---

## 🔵 INFRASTRUCTURE & SYSTEM — Foundation for Scale

These features are necessary for platform health and scalability but don't directly unlock segments.

| Priority | Feature | Type | Segments Impacted | Strategic Impact | Status in Backlog |
| --- | --- | --- | --- | --- | --- |
| **P2** | Rate limiting | Need Implement | ALL segments | Prevent abuse and manage API costs. Becomes critical as user base grows. Risk: high-volume users could drive up costs uncontrollably. | ✅ In current backlog as "Rate limiting" |
| **P2** | Multi-message handling | Enhancement | ALL segments | Handle multiple messages sent simultaneously (queue, parallel processing, or blocking). Currently causes context loss and confusion. | ✅ In current backlog as "Multi-message handling" |
| **P3** | Reminder feature improvements | Enhancement | ALL segments | Review and improve reminder system for pending tasks and open conversations. Currently works but could be more effective. | ✅ In current backlog as "Reminder feature improvements" |
| **P3** | Non-Pinto question handling | Need Implement | ALL segments | Politely decline or escalate questions unrelated to Pinto. Low urgency but improves professionalism. | ✅ In current backlog as "Non-Pinto question handling" |

---

## 🟣 ONBOARDING & SUPPORT — User Acquisition & Retention

These features improve onboarding experience and reduce support burden.

| Priority | Feature | Type | Segments Impacted | Strategic Impact | Status in Backlog |
| --- | --- | --- | --- | --- | --- |
| **P3** | Answer general accounting questions (Knowledge Base) | Need Implement | ALL segments (especially Freelancers, F&B, Beauty, Retail) | Enable Agent to answer "What is VAT?" or "How do I record an expense?" Reduces support burden for basic questions. | ✅ In current backlog as "Answer general accounting questions (Knowledge Base)" |
| **P3** | Knowledge base for product explanation | Enhancement | ALL segments | Improve how Agent explains what Pinto does and how to use it. Better onboarding = higher activation. | ✅ In current backlog as "Knowledge base for product explanation" |
| **P3** | Client support escalation (Human in the loop) | Need Implement | ALL segments | Enable users to escalate support questions to human agent. Safety net for when AI fails. | ✅ In current backlog as "Client support escalation (Human in the loop)" |
| **P4** | Automated Wafeq/Zoho account creation | Need Implement | ALL segments | Auto-create accounting system account during onboarding (not just OAuth). Reduces onboarding friction but low urgency. | ✅ In current backlog as "Automated Wafeq/Zoho account creation" |

---

## 📊 TASK MANAGEMENT & DASHBOARDS

These features improve internal operations and user task resolution.

| Priority | Feature | Type | Segments Impacted | Strategic Impact | Status in Backlog |
| --- | --- | --- | --- | --- | --- |
| **P2** | Task creation fallback (Invoice) | Need Implement | ALL segments | Create tasks when Agent cannot complete invoice automatically. Currently missing, causes lost invoices. | ✅ In current backlog as "Task creation fallback (Invoice)" |
| **P2** | Task creation fallback (Payment) | Need Implement | ALL segments | Create tasks when Agent cannot process payment automatically. Currently missing, causes lost payments. | ✅ In current backlog as "Task creation fallback (Payment)" |
| **P2** | Task creation fallback (Report) | Need Implement | ALL segments | Create tasks when Agent cannot generate report automatically. Currently missing, users get stuck. | ✅ In current backlog as "Task creation fallback (Report)" |
| **P3** | Retool task management improvements | Need Implement | Internal operations | Enhance task workflows, status tracking, accountant actions. Improves accountant efficiency. | ✅ In current backlog as "Retool task management improvements" |
| **P3** | Web app task management improvements | Need Implement | ALL segments | Improve how users view, respond to, and resolve tasks. Better UX = faster resolution. | ✅ In current backlog as "Web app task management improvements" |
| **P3** | Dashboard redesign with exports | Need Implement | ALL segments (especially Digital Agencies, Professional Services, Startups) | Redesign dashboard UI and add export functionality. Nice improvement but not critical. | ✅ In current backlog as "Dashboard redesign with exports" |
| **P3** | Actionable dashboard features | Need Implement | ALL segments | Add filters (date, vendor, status) and quick action buttons. Makes dashboard more useful. | ✅ In current backlog as "Actionable dashboard features" |

---

## 📋 Summary: Recommended Implementation Order

### Phase 1 (Months 1-3): Critical Blockers
1. **Bank statement processing** — Universal gap, unlocks 13/14 segments
2. **Project/job tagging** — Unlocks Digital Agencies, Construction, Professional Services
3. **Multi-currency partial payments** — Unblocks Trading, Agencies
4. **Refunds and returns** — Unblocks E-commerce

### Phase 2 (Months 3-6): High-Value Enhancements
5. **Custom accounting queries** — Dramatically improves UX across all segments
6. **Query accounting system data** — Enables powerful conversational queries
7. **Un-earned revenue invoices** — Critical for agencies, professional services
8. **Prepaid/accrual expenses** — Critical for trading, professional services
9. **Backend messaging with context** — Fixes conversation coherence issues
10. **Thread management after invoice creation** — Removes major UX friction

### Phase 3 (Months 6-9): Reliability & Polish
11. **OCR validation improvements** — Reduces task escalations
12. **Chart of accounts history** — Speeds up expense recording
13. **Task creation fallback (Invoice, Payment, Report)** — Prevents lost transactions
14. **Rate limiting** — Platform health as user base grows
15. **Multi-message handling** — Fixes context loss

### Phase 4 (Months 9-12): Nice-to-Haves
16. **Report delivery improvements**
17. **Dashboard redesign**
18. **Knowledge base features**
19. **Custom reports (Human in the loop)**
20. All remaining P3/P4 features

---

## 🎯 Strategic Insights

### Universal Gaps (Fix These First)
- **Bank reconciliation** appears in 13/14 segments
- **E-invoice compliance** appears in 11/14 segments (not in current backlog but should be roadmapped)
- **Payment gateway integration** appears in 10/14 segments (not in current backlog but should be roadmapped)

### Segment Unlock Map
| Feature | Unlocks | Coverage Lift |
| --- | --- | --- |
| Project/job tagging | Digital Agencies, Construction, Professional Services, Auto, Transport | 5 segments |
| Bank reconciliation | ALL 14 segments | Universal |
| Multi-currency payments | Trading, Agencies, E-commerce, Real Estate, Startups | 5 segments |
| Refunds/returns | E-commerce, Retail, F&B | 3 segments (including 1 five-star) |

### Quick Wins (Enhancements vs. New Build)
**Enhancements** (faster to ship):
- Multi-currency partial payment handling
- Thread management after invoice creation
- OCR validation improvements
- Chart of accounts history matching
- Manual entry flow improvements

**Need Implement** (longer build):
- Bank statement processing
- Project/job tagging
- Refunds and returns
- Custom accounting queries
- Query accounting system data

### Revenue Impact Priority
**Highest revenue impact** (unlocks $375K+ segments):
1. Bank reconciliation (Trading ⭐⭐⭐⭐⭐, Professional Services ⭐⭐⭐⭐)
2. Project/job tagging (Digital Agencies ⭐⭐⭐⭐⭐, Construction ⭐⭐⭐⭐⭐)
3. Multi-currency payments (Trading ⭐⭐⭐⭐⭐)
4. Un-earned revenue invoices (Agencies ⭐⭐⭐⭐⭐, Professional Services ⭐⭐⭐⭐)

---

## ⚠️ Notable Omissions from Current Backlog

These strategic features appear in segment gaps but are NOT in the current backlog. Consider adding to roadmap:

1. **E-invoice compliance** — Appears in 11/14 segments, critical for Professional Services, Trading, Real Estate
2. **Payment gateway integrations (Stripe, PayTabs, Telr)** — Appears in 10/14 segments
3. **E-commerce platform integrations (Salla, Shopify, Noon)** — Critical blocker for E-commerce segment (5-star priority)
4. **Recurring invoice automation** — Requested by Digital Agencies, Freelancers, Professional Services
5. **Multi-entity/branch support** — Critical for Trading, Startups, Professional Services
6. **Corporate tax reporting** — Critical for all $375K+ segments
7. **Burn rate reporting** — Critical for Tech Startups segment

These should be scoped and added to the roadmap based on segment focus.
