---
title: "creat vendor manual inpute"
publish: true
---

# creat vendor manual inpute

Source: `workflows/remote/dev_4/subagents/Expense agent V3/tools/creat vendor manual inpute.json`

Purpose:
Creates a new vendor during a manual expense-entry conversation.

Technical flow:
The workflow uses chat memory, checks for existing matches via HTTP, loops through candidates, waits when needed for controlled retries, creates the vendor if no safe match exists, and writes the matched/created vendor identity into internal tables for reuse by the expense agent.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  Chat_Memory_Manager_2["Chat Memory Manager"]
  Edit_Fields1_3["Edit Fields1"]
  Send_message_on_whatsapp_4["Send message on whatsapp"]
  match_as_check_5["match as check"]
  Postgres_Chat_Memory_6["Postgres Chat Memory"]
  Code_7["Code"]
  Wait1_8["Wait1"]
  Loop_Over_Items_9["Loop Over Items"]
  If_10["If"]
  0050502_11["0050502"]
  0050503_12["0050503"]
  create_13["create"]
  Upsert_row_s_4_14["Upsert row(s)4"]
  Edit_Fields_15["Edit Fields"]
  Upsert_row_s__16["Upsert row(s)"]
  Edit_Fields2_17["Edit Fields2"]
  Merge_18["Merge"]
  When_Executed_by_Another_Workflow_1 -->|main| Send_message_on_whatsapp_4
  When_Executed_by_Another_Workflow_1 -->|main | item 2| create_13
  match_as_check_5 -->|main| Code_7
  match_as_check_5 -->|main | out 2| 0050502_11
  Postgres_Chat_Memory_6 -.->|ai_memory| Chat_Memory_Manager_2
  Code_7 -->|main| If_10
  Code_7 -->|main | out 2| 0050503_12
  Wait1_8 -->|main| Loop_Over_Items_9
  Loop_Over_Items_9 -->|main | out 2| match_as_check_5
  If_10 -->|main| Upsert_row_s_4_14
  If_10 -->|main | item 2| Edit_Fields2_17
  If_10 -->|main | out 2| Wait1_8
  create_13 -->|main| Loop_Over_Items_9
  create_13 -->|main | out 2| Edit_Fields_15
  Upsert_row_s_4_14 -->|main| Merge_18
  Edit_Fields2_17 -->|main| Upsert_row_s__16
  Upsert_row_s__16 -->|main| Merge_18
  Merge_18 -->|main| Edit_Fields1_3
```
