---
title: "Record expense Manuel inpute"
publish: true
---

# Record expense Manuel inpute

Source: `workflows/remote/dev_4/subagents/Expense agent V3/tools/Record expense Manuel inpute.json`

Purpose:
Records an expense when the data was collected manually through conversation instead of OCR.

Technical flow:
The workflow receives normalized expense fields from the agent, shapes the API payload in JavaScript, calls the external expense-creation endpoint, updates the thread status, stores the result in Postgres, and sends the user a WhatsApp confirmation or follow-up message.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  Record_Unpaid_expense_2["Record Unpaid expense"]
  Send_message_on_whatsapp_3["Send message on whatsapp"]
  0050601_4["0050601"]
  Code_in_JavaScript_5["Code in JavaScript"]
  Update_thread_status_6["Update thread status"]
  Insert_or_update_rows_in_a_table_7["Insert or update rows in a table"]
  When_Executed_by_Another_Workflow_1 -->|main| Send_message_on_whatsapp_3
  When_Executed_by_Another_Workflow_1 -->|main | item 2| Code_in_JavaScript_5
  Record_Unpaid_expense_2 -->|main| Insert_or_update_rows_in_a_table_7
  Code_in_JavaScript_5 -->|main| Record_Unpaid_expense_2
  Insert_or_update_rows_in_a_table_7 -->|main| Update_thread_status_6
```
