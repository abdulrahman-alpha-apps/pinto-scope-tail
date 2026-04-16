---
title: "Change routing state"
publish: true
---

# Change routing state

Source: `workflows/remote/dev_4/subagents/Invoice agent/tools/Change routing state.json`

Purpose:
Transfers the conversation from the current agent to another domain agent.

Technical flow:
The workflow receives a requested target state, writes the new routing state into Postgres, updates the thread status in the external app, optionally sends the user a confirmation message, and then executes the target workflow for expense, payment, invoice, or reporting. It also protects against invalid targets and unnecessary looping.

Why it is important:
This is the router-handoff mechanism exposed as a tool so an agent can intentionally move a thread to a more appropriate specialist.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  Update_thread_status_2["Update thread status"]
  Invoice_agent_3["Invoice agent"]
  Report_agent_4["Report agent"]
  Switch_5["Switch"]
  Payment_agent_6["Payment agent"]
  Insert_or_update_new_state_7["Insert or update new state"]
  Expense_agent2_8["Expense agent2"]
  Edit_Fields_9["Edit Fields"]
  Update_thread_status1_10["Update thread status1"]
  Send_message_on_whatsapp_11["Send message on whatsapp"]
  If_12["If"]
  Not_looping_over_13["Not looping over"]
  Edit_Fields1_14["Edit Fields1"]
  Wrong_target_15["Wrong target"]
  Edit_Fields2_16["Edit Fields2"]
  When_Executed_by_Another_Workflow_1 -->|main| Not_looping_over_13
  Switch_5 -->|main| Expense_agent2_8
  Switch_5 -->|main | out 2| Payment_agent_6
  Switch_5 -->|main | out 3| Invoice_agent_3
  Switch_5 -->|main | out 4| Report_agent_4
  Switch_5 -->|main | out 5| Wrong_target_15
  Insert_or_update_new_state_7 -->|main| If_12
  Expense_agent2_8 -->|main| Edit_Fields_9
  Payment_agent_6 -->|main| Edit_Fields_9
  Invoice_agent_3 -->|main| Edit_Fields_9
  Report_agent_4 -->|main| Edit_Fields_9
  Update_thread_status1_10 -->|main| Send_message_on_whatsapp_11
  Send_message_on_whatsapp_11 -->|main| Switch_5
  If_12 -->|main| Update_thread_status1_10
  Not_looping_over_13 -->|main| Insert_or_update_new_state_7
  Not_looping_over_13 -->|main | out 2| Edit_Fields2_16
  Wrong_target_15 -->|main| Edit_Fields1_14
```
