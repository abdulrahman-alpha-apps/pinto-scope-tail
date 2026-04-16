---
title: "Onboarding agent"
publish: true
---

# Onboarding agent

Source: `workflows/remote/dev_4/subagents/Onboarding agent/Onboarding agent.json`

Purpose:
This is the patient onboarding or account onboarding specialist. It guides the user through connecting accounting systems and updating profile/setup information.

Technical flow:
The workflow is triggered by the router, checks whether onboarding has already been triggered for the thread, and then uses an AI agent with tools for generating Wafeq and Zoho auth URLs and for updating user information. It logs onboarding events and ships them to BigQuery through the shared analytics workflow.

Outputs:
It returns user-facing WhatsApp messages and structured onboarding progress updates back into the shared state model.

Tools:
- [Onboarding agent tools](/Users/pemo/Desktop/n8n Pinto/explanations/router%20agent/Subagents/Onboarding%20agent/tools)

## Diagram

```mermaid
flowchart TD
  AI_Agent_1["AI Agent"]
  Get_Wafeq_Auth_Url_2["Get Wafeq Auth Url"]
  Update_user_info_3["Update user info"]
  Send_message_on_whatsapp_4["Send message on whatsapp"]
  When_Executed_by_Another_Workflow_5["When Executed by Another Workflow"]
  OpenAI_Chat_Model_6["OpenAI Chat Model"]
  Get_Zoho_Auth_Url_7["Get Zoho Auth Url"]
  Postgres_Chat_Memory_8["Postgres Chat Memory"]
  Edit_Fields_9["Edit Fields"]
  Event_Aggregator_10["Event Aggregator"]
  Events_Exist__11["Events Exist?"]
  Check_Onboarding_Triggered_12["Check Onboarding Triggered"]
  Aggregate_13["Aggregate"]
  Call__Events_to_bigQuery__14["Call 'Events to bigQuery'"]
  Get_Wafeq_Auth_Url_2 -.->|ai_tool| AI_Agent_1
  AI_Agent_1 -->|main| Event_Aggregator_10
  Update_user_info_3 -.->|ai_tool| AI_Agent_1
  When_Executed_by_Another_Workflow_5 -->|main| Check_Onboarding_Triggered_12
  Send_message_on_whatsapp_4 -->|main| Edit_Fields_9
  OpenAI_Chat_Model_6 -.->|ai_languageModel| AI_Agent_1
  Get_Zoho_Auth_Url_7 -.->|ai_tool| AI_Agent_1
  Postgres_Chat_Memory_8 -.->|ai_memory| AI_Agent_1
  Event_Aggregator_10 -->|main| Events_Exist__11
  Events_Exist__11 -->|main| Call__Events_to_bigQuery__14
  Events_Exist__11 -->|main | out 2| Send_message_on_whatsapp_4
  Check_Onboarding_Triggered_12 -->|main| AI_Agent_1
  Aggregate_13 -->|main| Send_message_on_whatsapp_4
  Call__Events_to_bigQuery__14 -->|main| Aggregate_13
```
