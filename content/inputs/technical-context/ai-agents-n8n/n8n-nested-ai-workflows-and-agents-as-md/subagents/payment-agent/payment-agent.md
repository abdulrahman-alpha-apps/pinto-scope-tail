---
title: "Payment agent"
publish: true
---

# Payment agent

Source: `workflows/remote/dev_4/subagents/Payment agent/Payment agent.json`

Purpose:
This is the payment specialist. It handles payment extraction, contact matching/creation, payment-account selection, invoice or bill lookup, cash payment posting, and fallback escalation.

Technical flow:
The workflow is router-triggered, writes an initial state, and then lets an AI agent choose among tools for extracting payment info from CSV, extracting bank statements, matching contacts, creating contacts, fetching payment accounts, fetching chart-of-accounts and invoice/bill details, creating cash payments, or changing routing state. Like the invoice agent, it maintains timeout state and sends events to BigQuery.

Side effects:
It sends WhatsApp responses, updates thread state in Postgres/Data Tables, and returns a normalized payment-operation result to the router layer.

Tools:
- [Payment agent tools](/Users/pemo/Desktop/n8n Pinto/explanations/router%20agent/Subagents/Payment%20agent/tools)

## Diagram

```mermaid
flowchart TD
  OpenAI_Chat_Model_1["OpenAI Chat Model"]
  Extract_Payment_Info_from_CSV_File_2["Extract Payment Info from CSV File"]
  Extract_Bank_Statement_File_3["Extract Bank Statement File"]
  AI_Agent1_4["AI Agent1"]
  When_Executed_by_Another_Workflow_5["When Executed by Another Workflow"]
  Send_message_on_whatsapp_6["Send message on whatsapp"]
  Create_bank_statements_7["Create bank statements"]
  Change_routing_state_8["Change routing state"]
  Match_contact_9["Match contact"]
  Create_contact_10["Create contact"]
  update_stats_11["update stats"]
  output_12["output"]
  Get_Payment_Account_13["Get Payment Account"]
  Get_chart_of_accounts_14["Get chart of accounts"]
  Insert_or_update_new_state_15["Insert or update new state"]
  Event_Aggregator_16["Event Aggregator"]
  Events_Exist__17["Events Exist?"]
  Aggregate_18["Aggregate"]
  Check_Payment_Triggered_19["Check Payment Triggered"]
  Get_invoice_bill_details_20["Get invoice/bill details"]
  Exchange_rate_21["Exchange-rate"]
  Set_Timeout_State_22["Set Timeout State"]
  Trigger_Timeout_Monitor_23["Trigger Timeout Monitor"]
  Clear_Timeout_State_24["Clear Timeout State"]
  Create_contact1_25["Create contact1"]
  Create_Cash_Payment1_26["Create Cash Payment1"]
  Simple_Memory_27["Simple Memory"]
  Call__Events_to_bigQuery__28["Call 'Events to bigQuery'"]
  Payment_Fallback_29["Payment Fallback"]
  OpenAI_Chat_Model_1 -.->|ai_languageModel| AI_Agent1_4
  Extract_Payment_Info_from_CSV_File_2 -.->|ai_tool| AI_Agent1_4
  Extract_Bank_Statement_File_3 -.->|ai_tool| AI_Agent1_4
  When_Executed_by_Another_Workflow_5 -->|main| Set_Timeout_State_22
  AI_Agent1_4 -->|main| Event_Aggregator_16
  Create_bank_statements_7 -.->|ai_tool| AI_Agent1_4
  Change_routing_state_8 -.->|ai_tool| AI_Agent1_4
  Match_contact_9 -.->|ai_tool| AI_Agent1_4
  Create_contact_10 -.->|ai_tool| AI_Agent1_4
  update_stats_11 -.->|ai_tool| AI_Agent1_4
  Send_message_on_whatsapp_6 -->|main| output_12
  Get_Payment_Account_13 -.->|ai_tool| AI_Agent1_4
  Get_chart_of_accounts_14 -.->|ai_tool| AI_Agent1_4
  Insert_or_update_new_state_15 -->|main| AI_Agent1_4
  Event_Aggregator_16 -->|main| Events_Exist__17
  Events_Exist__17 -->|main| Call__Events_to_bigQuery__28
  Events_Exist__17 -->|main | out 2| Clear_Timeout_State_24
  Aggregate_18 -->|main| Clear_Timeout_State_24
  Check_Payment_Triggered_19 -->|main| Insert_or_update_new_state_15
  Get_invoice_bill_details_20 -.->|ai_tool| AI_Agent1_4
  Exchange_rate_21 -.->|ai_tool| AI_Agent1_4
  Set_Timeout_State_22 -->|main| Trigger_Timeout_Monitor_23
  Trigger_Timeout_Monitor_23 -->|main| Check_Payment_Triggered_19
  Clear_Timeout_State_24 -->|main| Send_message_on_whatsapp_6
  Create_contact1_25 -.->|ai_tool| AI_Agent1_4
  Create_Cash_Payment1_26 -.->|ai_tool| AI_Agent1_4
  Simple_Memory_27 -.->|ai_memory| AI_Agent1_4
  Call__Events_to_bigQuery__28 -->|main| Aggregate_18
  Payment_Fallback_29 -.->|ai_tool| AI_Agent1_4
```
