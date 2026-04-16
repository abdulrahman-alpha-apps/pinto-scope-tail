---
title: "Expense agent"
publish: true
---

# Expense agent

Source: `workflows/remote/dev_4/subagents/Expense agent V3/Expense agent V3.json`

Purpose:
This is the specialist agent for expense capture. It manages both OCR-driven receipt/invoice processing and manual expense entry conversations.

Technical flow:
The workflow is invoked by another workflow, reads the current expense thread state from data tables, and decides whether the file is being processed for the first time. If the input is file-led, it calls the automated OCR tool and then hands the extracted result into the OCR agent. If the input is conversational/manual, it loads chart-of-account and payment-account context and uses the Manual Input Agent to collect missing fields.

Agent/tool orchestration:
The agent exposes tools for recording the expense, matching or creating a vendor, selecting an account, updating routing state, and fallback handling. It also writes router state back into shared tables so the root router knows the current stage of the thread.

Outputs and side effects:
It emits events for analytics, sends WhatsApp updates, records expense metadata in tables, and returns a normalized agent response back to the router layer.

Tools:
- [Expense agent tools](/Users/pemo/Desktop/n8n Pinto/explanations/router%20agent/Subagents/Expense%20agent/tools)

## Diagram

```mermaid
flowchart TD
  Get_Data1_1["Get Data1"]
  Get_Data_2["Get Data"]
  Edit_Fields_3["Edit Fields"]
  Upsert_row_s__4["Upsert row(s)"]
  Limit_5["Limit"]
  Record_expense_6["Record expense"]
  Change_routing_state_7["Change routing state"]
  Fallback_8["Fallback"]
  update_stats_9["update stats"]
  Get_Payment_Accounts_10["Get Payment Accounts"]
  Match_Vendor_11["Match Vendor"]
  creat_vendor_tool_12["creat vendor tool"]
  OpenAI_Chat_Model_13["OpenAI Chat Model"]
  Profom_OCR_and_update_log_14["Profom OCR and update log"]
  Call__OCR_automated_tool__15["Call 'OCR automated tool'"]
  When_Executed_by_Another_Workflow_16["When Executed by Another Workflow"]
  Get_Payment_Accounts1_17["Get Payment Accounts1"]
  Limit3_18["Limit3"]
  Get_charts_of_account1_19["Get charts of account1"]
  Record_expense1_20["Record expense1"]
  Event_Aggregator_V_21["Event Aggregator V"]
  Send_to_Event_Collector1_22["Send to Event Collector1"]
  Events_Exist_1_23["Events Exist?1"]
  Match_Vendor1_24["Match Vendor1"]
  Message_back1_25["Message back1"]
  creat_vendor_tool1_26["creat vendor tool1"]
  Send_message_on_whatsapp1_27["Send message on whatsapp1"]
  Get_Data2_28["Get Data2"]
  Limit1_29["Limit1"]
  file_first_time1_30["file first time1"]
  OCR_Agent_31["OCR Agent"]
  Manual_input_Agent_32["Manual input Agent"]
  update_router_33["update router"]
  Redis_Chat_Memory_34["Redis Chat Memory"]
  Get_Data_2 -->|main| file_first_time1_30
  Get_Data1_1 -->|main| Get_Data2_28
  Limit_5 -->|main| OCR_Agent_31
  Fallback_8 -.->|ai_tool| OCR_Agent_31
  Fallback_8 -.->|ai_tool | item 2| Manual_input_Agent_32
  Change_routing_state_7 -.->|ai_tool| OCR_Agent_31
  Change_routing_state_7 -.->|ai_tool | item 2| Manual_input_Agent_32
  update_stats_9 -.->|ai_tool| OCR_Agent_31
  Record_expense_6 -.->|ai_tool| OCR_Agent_31
  OpenAI_Chat_Model_13 -.->|ai_languageModel| OCR_Agent_31
  OpenAI_Chat_Model_13 -.->|ai_languageModel | item 2| Manual_input_Agent_32
  Call__OCR_automated_tool__15 -->|main| Get_Data_2
  When_Executed_by_Another_Workflow_16 -->|main| update_router_33
  Event_Aggregator_V_21 -->|main| Events_Exist_1_23
  Send_to_Event_Collector1_22 -->|main| Limit3_18
  Events_Exist_1_23 -->|main| Send_to_Event_Collector1_22
  Events_Exist_1_23 -->|main | out 2| Limit3_18
  Limit3_18 -->|main| Send_message_on_whatsapp1_27
  Get_charts_of_account1_19 -.->|ai_tool| Manual_input_Agent_32
  Record_expense1_20 -.->|ai_tool| Manual_input_Agent_32
  Match_Vendor1_24 -.->|ai_tool| Manual_input_Agent_32
  creat_vendor_tool1_26 -.->|ai_tool| Manual_input_Agent_32
  Send_message_on_whatsapp1_27 -->|main| Message_back1_25
  Get_Data2_28 -->|main| Limit1_29
  Limit1_29 -->|main| Manual_input_Agent_32
  file_first_time1_30 -->|main| Call__OCR_automated_tool__15
  file_first_time1_30 -->|main | out 2| Limit_5
  file_first_time1_30 -->|main | out 3| Get_Data1_1
  OCR_Agent_31 -->|main| Event_Aggregator_V_21
  Manual_input_Agent_32 -->|main| Event_Aggregator_V_21
  update_router_33 -->|main| Get_Data_2
  Redis_Chat_Memory_34 -.->|ai_memory| Manual_input_Agent_32
  Redis_Chat_Memory_34 -.->|ai_memory | item 2| OCR_Agent_31
```
