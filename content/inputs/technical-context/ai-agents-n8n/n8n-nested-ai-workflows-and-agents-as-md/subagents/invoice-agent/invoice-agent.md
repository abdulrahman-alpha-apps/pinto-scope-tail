---
title: "Invoice agent"
publish: true
---

# Invoice agent

Source: `workflows/remote/dev_4/subagents/Invoice agent/Invoice agent.json`

Purpose:
This is the invoice specialist. It drives invoice creation, item/customer matching, invoice delivery, and cross-agent routing when the conversation needs to move elsewhere.

Technical flow:
The workflow supports both direct chat entry and router-triggered entry. It initializes thread state, loads memory, and lets an AI agent decide which tool to call: create items, match items, match or create the customer, select accounts, create the invoice, deliver the invoice, or change routing state. It also aggregates execution events and sends them to BigQuery analytics.

State management:
The workflow sets a timeout state and triggers a timeout monitor workflow. If the conversation completes before the timeout, it clears the state. This prevents silent stalls in invoice flows.

Tools:
- [Invoice agent tools](/Users/pemo/Desktop/n8n Pinto/explanations/router%20agent/Subagents/Invoice%20agent/tools)

## Diagram

```mermaid
flowchart TD
  AI_Agent_1["AI Agent"]
  OpenAI_Chat_Model_2["OpenAI Chat Model"]
  Submit_invoice_3["Submit invoice"]
  When_chat_message_received_4["When chat message received"]
  Send_message_on_whatsapp_5["Send message on whatsapp"]
  When_Executed_by_Another_Workflow_6["When Executed by Another Workflow"]
  update_stats_7["update stats"]
  Create_Items_8["Create Items"]
  Match_items_9["Match items"]
  Match_customer_10["Match customer"]
  Create_customer_11["Create customer"]
  Change_routing_state_12["Change routing state"]
  output_13["output"]
  Insert_or_update_new_state_14["Insert or update new state"]
  Postgres_Chat_Memory_15["Postgres Chat Memory"]
  Get_row_s__16["Get row(s)"]
  Event_Aggregator_17["Event Aggregator"]
  Events_Exist__18["Events Exist?"]
  Send_to_Event_Collector_19["Send to Event Collector"]
  Check_Invoice_Triggered_20["Check Invoice Triggered"]
  Aggregate_21["Aggregate"]
  Datadog_HTTP_Request_22["Datadog HTTP Request"]
  Get_Accounts_Tool_23["Get Accounts Tool"]
  Upsert_row_s__24["Upsert row(s)"]
  Invoice_Delivery_Tool_25["Invoice Delivery Tool"]
  Simple_Memory_26["Simple Memory"]
  AI_Agent1_27["AI Agent1"]
  OpenAI_Chat_Model1_28["OpenAI Chat Model1"]
  Submit_invoice1_29["Submit invoice1"]
  Send_message_on_whatsapp1_30["Send message on whatsapp1"]
  update_stats1_31["update stats1"]
  Create_Items1_32["Create Items1"]
  Match_items1_33["Match items1"]
  Match_customer1_34["Match customer1"]
  Create_customer1_35["Create customer1"]
  Change_routing_state1_36["Change routing state1"]
  output1_37["output1"]
  Insert_or_update_new_state1_38["Insert or update new state1"]
  Postgres_Chat_Memory1_39["Postgres Chat Memory1"]
  Get_row_s_1_40["Get row(s)1"]
  Event_Aggregator1_41["Event Aggregator1"]
  Events_Exist_1_42["Events Exist?1"]
  Check_Invoice_Triggered1_43["Check Invoice Triggered1"]
  Aggregate1_44["Aggregate1"]
  Get_Accounts_Tool1_45["Get Accounts Tool1"]
  Upsert_row_s_1_46["Upsert row(s)1"]
  Invoice_Delivery_Tool1_47["Invoice Delivery Tool1"]
  Simple_Memory1_48["Simple Memory1"]
  Set_Timeout_State_49["Set Timeout State"]
  Trigger_Timeout_Monitor_50["Trigger Timeout Monitor"]
  Clear_Timeout_State_51["Clear Timeout State"]
  Call__Events_to_bigQuery__52["Call 'Events to bigQuery'"]
  OpenAI_Chat_Model_2 -.->|ai_languageModel| AI_Agent_1
  Submit_invoice_3 -.->|ai_tool| AI_Agent_1
  When_chat_message_received_4 -->|main| Insert_or_update_new_state_14
  AI_Agent_1 -->|main| Event_Aggregator_17
  When_Executed_by_Another_Workflow_6 -->|main| Set_Timeout_State_49
  update_stats_7 -.->|ai_tool| AI_Agent_1
  Create_Items_8 -.->|ai_tool| AI_Agent_1
  Send_message_on_whatsapp_5 -->|main| output_13
  Match_items_9 -.->|ai_tool| AI_Agent_1
  Match_customer_10 -.->|ai_tool| AI_Agent_1
  Create_customer_11 -.->|ai_tool| AI_Agent_1
  Change_routing_state_12 -.->|ai_tool| AI_Agent_1
  Insert_or_update_new_state_14 -->|main| Get_row_s__16
  Get_row_s__16 -->|main| Upsert_row_s__24
  Event_Aggregator_17 -->|main| Events_Exist__18
  Events_Exist__18 -->|main| Send_to_Event_Collector_19
  Events_Exist__18 -->|main | out 2| Send_message_on_whatsapp_5
  Send_to_Event_Collector_19 -->|main| Aggregate_21
  Check_Invoice_Triggered_20 -->|main| Insert_or_update_new_state_14
  Aggregate_21 -->|main| Send_message_on_whatsapp_5
  Get_Accounts_Tool_23 -.->|ai_tool| AI_Agent_1
  Upsert_row_s__24 -->|main| AI_Agent_1
  Invoice_Delivery_Tool_25 -.->|ai_tool| AI_Agent_1
  Simple_Memory_26 -.->|ai_memory| AI_Agent_1
  AI_Agent1_27 -->|main| Event_Aggregator1_41
  OpenAI_Chat_Model1_28 -.->|ai_languageModel| AI_Agent1_27
  Submit_invoice1_29 -.->|ai_tool| AI_Agent1_27
  Send_message_on_whatsapp1_30 -->|main| output1_37
  update_stats1_31 -.->|ai_tool| AI_Agent1_27
  Create_Items1_32 -.->|ai_tool| AI_Agent1_27
  Match_items1_33 -.->|ai_tool| AI_Agent1_27
  Match_customer1_34 -.->|ai_tool| AI_Agent1_27
  Create_customer1_35 -.->|ai_tool| AI_Agent1_27
  Change_routing_state1_36 -.->|ai_tool| AI_Agent1_27
  Insert_or_update_new_state1_38 -->|main| Get_row_s_1_40
  Get_row_s_1_40 -->|main| Upsert_row_s_1_46
  Event_Aggregator1_41 -->|main| Events_Exist_1_42
  Events_Exist_1_42 -->|main| Call__Events_to_bigQuery__52
  Events_Exist_1_42 -->|main | out 2| Clear_Timeout_State_51
  Check_Invoice_Triggered1_43 -->|main| Insert_or_update_new_state1_38
  Aggregate1_44 -->|main| Clear_Timeout_State_51
  Get_Accounts_Tool1_45 -.->|ai_tool| AI_Agent1_27
  Upsert_row_s_1_46 -->|main| AI_Agent1_27
  Invoice_Delivery_Tool1_47 -.->|ai_tool| AI_Agent1_27
  Simple_Memory1_48 -.->|ai_memory| AI_Agent1_27
  Set_Timeout_State_49 -->|main| Trigger_Timeout_Monitor_50
  Trigger_Timeout_Monitor_50 -->|main| Check_Invoice_Triggered1_43
  Clear_Timeout_State_51 -->|main| Send_message_on_whatsapp1_30
  Call__Events_to_bigQuery__52 -->|main| Aggregate1_44
```
