---
title: "Fallback dummy"
publish: true
---

# Fallback dummy

Source: `workflows/remote/dev_4/subagents/Expense agent V3/tools/Fallback dummy.json`

Purpose:
Fallback path for expense conversations that cannot continue cleanly.

Technical flow:
The workflow updates the thread status, attempts to create a follow-up task in an external web app, stores the fallback state in Postgres, and records or resets chat memory so the thread can be recovered later by a human or another workflow.

When it is used:
This tool is called when the expense agent cannot confidently complete the requested action or needs to escalate the conversation.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  Update_thread_status_2["Update thread status"]
  Chat_Memory_Manager_3["Chat Memory Manager"]
  Handle_creating_a_task_4["Handle creating a task"]
  new_task_were_created_in_web_app_5["new task were created in web app"]
  Unable_to_create_task_6["Unable to create task"]
  Update_thread_status1_7["Update thread status1"]
  Insert_or_update_rows_in_a_table_8["Insert or update rows in a table"]
  Postgres_Chat_Memory_9["Postgres Chat Memory"]
  Edit_Fields_10["Edit Fields"]
  When_Executed_by_Another_Workflow_1 -->|main| Edit_Fields_10
  Update_thread_status_2 -->|main| new_task_were_created_in_web_app_5
  Update_thread_status_2 -->|main | item 2| Insert_or_update_rows_in_a_table_8
  Handle_creating_a_task_4 -->|main| Update_thread_status_2
  Handle_creating_a_task_4 -->|main | out 2| Update_thread_status1_7
  Update_thread_status1_7 -->|main| Unable_to_create_task_6
  Update_thread_status1_7 -->|main | item 2| Insert_or_update_rows_in_a_table_8
  Postgres_Chat_Memory_9 -.->|ai_memory| Chat_Memory_Manager_3
```
