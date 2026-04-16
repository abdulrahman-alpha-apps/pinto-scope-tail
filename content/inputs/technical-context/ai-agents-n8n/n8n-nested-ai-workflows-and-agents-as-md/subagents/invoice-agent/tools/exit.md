---
title: "exit"
publish: true
---

# exit

Source: `workflows/remote/dev_4/subagents/Invoice agent/tools/exit.json`

Purpose:
Closes an invoice conversation cleanly.

Technical flow:
It updates the thread status, stores a terminal state in Postgres, and resets or records memory so the invoice agent does not continue treating the thread as active.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  Update_thread_status_2["Update thread status"]
  Chat_Memory_Manager_3["Chat Memory Manager"]
  Insert_or_update_rows_in_a_table_4["Insert or update rows in a table"]
  Simple_Memory_5["Simple Memory"]
  When_Executed_by_Another_Workflow_1 -->|main| Insert_or_update_rows_in_a_table_4
  Update_thread_status_2 -->|main| Chat_Memory_Manager_3
  Insert_or_update_rows_in_a_table_4 -->|main| Update_thread_status_2
  Simple_Memory_5 -.->|ai_memory| Chat_Memory_Manager_3
```
