---
title: "Get invoice_bill details"
publish: true
---

# Get invoice/bill details

Source: `workflows/remote/dev_4/subagents/Payment agent/tools/Get invoice_bill details.json`

Purpose:
Fetches the accounting-system details for the invoice or bill being paid.

Technical flow:
It calls an HTTP endpoint to fetch invoice or bill details, persists the normalized result into local state, and stores the context in chat memory so the payment agent can reason about the target document during posting or reconciliation.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  HTTP_Request_2["HTTP Request"]
  Chat_Memory_Manager_3["Chat Memory Manager"]
  Postgres_Chat_Memory_4["Postgres Chat Memory"]
  Edit_Fields_5["Edit Fields"]
  Insert_matched_contact_6["Insert matched contact"]
  When_Executed_by_Another_Workflow_1 -->|main| HTTP_Request_2
  HTTP_Request_2 -->|main| Insert_matched_contact_6
  Postgres_Chat_Memory_4 -.->|ai_memory| Chat_Memory_Manager_3
  Chat_Memory_Manager_3 -->|main| Edit_Fields_5
  Insert_matched_contact_6 -->|main| Chat_Memory_Manager_3
```
