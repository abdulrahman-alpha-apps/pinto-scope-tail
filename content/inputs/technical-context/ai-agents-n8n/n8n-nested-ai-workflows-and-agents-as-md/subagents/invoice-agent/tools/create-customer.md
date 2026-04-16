---
title: "Create customer"
publish: true
---

# Create customer

Source: `workflows/remote/dev_4/subagents/Invoice agent/tools/Create customer.json`

Purpose:
Creates a customer record for invoice generation.

Technical flow:
It checks for likely matches first, uses chat memory to preserve context, loops over candidate results, and calls the create-customer API only when needed. The final customer identity is written into internal data tables so the invoice agent can immediately continue.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  Chat_Memory_Manager_2["Chat Memory Manager"]
  Edit_Fields1_3["Edit Fields1"]
  Send_message_on_whatsapp_4["Send message on whatsapp"]
  creat_5["creat"]
  match_as_check_6["match as check"]
  Postgres_Chat_Memory_7["Postgres Chat Memory"]
  Code_8["Code"]
  Wait1_9["Wait1"]
  Loop_Over_Items_10["Loop Over Items"]
  If_11["If"]
  Edit_Fields_12["Edit Fields"]
  Edit_Fields2_13["Edit Fields2"]
  Upsert_row_s__14["Upsert row(s)"]
  When_Executed_by_Another_Workflow_1 -->|main| Send_message_on_whatsapp_4
  When_Executed_by_Another_Workflow_1 -->|main | item 2| creat_5
  Chat_Memory_Manager_2 -->|main| Upsert_row_s__14
  creat_5 -->|main| Loop_Over_Items_10
  creat_5 -->|main | out 2| Edit_Fields_12
  match_as_check_6 -->|main| Code_8
  Postgres_Chat_Memory_7 -.->|ai_memory| Chat_Memory_Manager_2
  Code_8 -->|main| If_11
  Wait1_9 -->|main| Loop_Over_Items_10
  Loop_Over_Items_10 -->|main | out 2| match_as_check_6
  If_11 -->|main| Chat_Memory_Manager_2
  If_11 -->|main | out 2| Wait1_9
  Upsert_row_s__14 -->|main| Edit_Fields1_3
```
