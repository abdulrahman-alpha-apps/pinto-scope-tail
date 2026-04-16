---
title: "Reporting agent"
publish: true
---

# Reporting agent

Source: `workflows/remote/dev_4/subagents/Reporting agent/Reporting agent.json`

Purpose:
Specialist agent for report generation and reporting-related routing.

Technical flow:
The workflow starts from router or chat entry, checks whether reporting has already been triggered for the thread, and then lets an AI agent choose among tools for getting accounts, generating reports, generating statement-of-account reports, matching contacts, updating stats, or changing routing state. It also aggregates analytics events and ships them to BigQuery.

Typical use:
This agent is where conversational reporting requests are translated into concrete report-generation API actions.

Tools:
- [Reporting agent tools](/Users/pemo/Desktop/n8n Pinto/explanations/router%20agent/Subagents/Reporting%20agent/tools)

## Diagram

```mermaid
flowchart TD
  AI_Agent_1["AI Agent"]
  OpenAI_Chat_Model_2["OpenAI Chat Model"]
  When_Executed_by_Another_Workflow_3["When Executed by Another Workflow"]
  Get_Accounts_4["Get Accounts"]
  When_chat_message_received_5["When chat message received"]
  update_stats_6["update stats"]
  Generate_Statement_of_account_Reports_7["Generate Statement of account Reports"]
  Generate_Reports_8["Generate Reports"]
  Send_message_on_whatsapp_9["Send message on whatsapp"]
  Change_routing_state_10["Change routing state"]
  Match_Contact_11["Match Contact"]
  Insert_or_update_new_state_12["Insert or update new state"]
  Postgres_Chat_Memory_13["Postgres Chat Memory"]
  Check_Report_Triggered_14["Check Report Triggered"]
  Event_Aggregator_15["Event Aggregator"]
  Events_Exist__16["Events Exist?"]
  Aggregate_17["Aggregate"]
  Call__Events_to_bigQuery__18["Call 'Events to bigQuery'"]
  OpenAI_Chat_Model_2 -.->|ai_languageModel| AI_Agent_1
  When_Executed_by_Another_Workflow_3 -->|main| Check_Report_Triggered_14
  Get_Accounts_4 -.->|ai_tool| AI_Agent_1
  When_chat_message_received_5 -->|main| Insert_or_update_new_state_12
  AI_Agent_1 -->|main| Event_Aggregator_15
  update_stats_6 -.->|ai_tool| AI_Agent_1
  Generate_Statement_of_account_Reports_7 -.->|ai_tool| AI_Agent_1
  Generate_Reports_8 -.->|ai_tool| AI_Agent_1
  Change_routing_state_10 -.->|ai_tool| AI_Agent_1
  Match_Contact_11 -.->|ai_tool| AI_Agent_1
  Insert_or_update_new_state_12 -->|main| AI_Agent_1
  Postgres_Chat_Memory_13 -.->|ai_memory| AI_Agent_1
  Check_Report_Triggered_14 -->|main| Insert_or_update_new_state_12
  Event_Aggregator_15 -->|main| Events_Exist__16
  Events_Exist__16 -->|main| Call__Events_to_bigQuery__18
  Events_Exist__16 -->|main | out 2| Send_message_on_whatsapp_9
  Aggregate_17 -->|main| Send_message_on_whatsapp_9
  Call__Events_to_bigQuery__18 -->|main| Aggregate_17
```
