---
title: "match vendor"
publish: true
---

# match vendor

Source: `workflows/remote/dev_4/subagents/Reporting agent/tools/match vendor.json`

Purpose:
Matches a vendor/contact for reporting-related operations.

Technical flow:
The workflow uses an AI agent and structured parser to evaluate candidate contact matches, records different match outcomes in data tables, keeps memory context for follow-up questions, and sends a WhatsApp message when the match needs user confirmation.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  AI_Agent_2["AI Agent"]
  OpenAI_Chat_Model_3["OpenAI Chat Model"]
  Structured_Output_Parser_4["Structured Output Parser"]
  Chat_Memory_Manager_5["Chat Memory Manager"]
  Postgres_Chat_Memory_6["Postgres Chat Memory"]
  Edit_Fields1_7["Edit Fields1"]
  Match_contact_8["Match contact"]
  Simple_Memory_9["Simple Memory"]
  Send_message_on_whatsapp_10["Send message on whatsapp"]
  0050401_11["0050401"]
  Simple_Memory1_12["Simple Memory1"]
  update_vendor_matching_data_13["update vendor matching data"]
  notEmpty_14["notEmpty"]
  update_vendor_matching_data1_15["update vendor matching data1"]
  Edit_Fields_16["Edit Fields"]
  update_vendor_matching_data2_17["update vendor matching data2"]
  update_vendor_matching_data3_18["update vendor matching data3"]
  Merge_19["Merge"]
  Merge1_20["Merge1"]
  When_Executed_by_Another_Workflow_1 -->|main| AI_Agent_2
  When_Executed_by_Another_Workflow_1 -->|main | item 2| Send_message_on_whatsapp_10
  OpenAI_Chat_Model_3 -.->|ai_languageModel| AI_Agent_2
  Structured_Output_Parser_4 -.->|ai_outputParser| AI_Agent_2
  Postgres_Chat_Memory_6 -.->|ai_memory| Chat_Memory_Manager_5
  AI_Agent_2 -->|main| notEmpty_14
  AI_Agent_2 -->|main | out 2| 0050401_11
  Match_contact_8 -.->|ai_tool| AI_Agent_2
  Simple_Memory1_12 -.->|ai_memory| AI_Agent_2
  update_vendor_matching_data_13 -->|main| Merge_19
  notEmpty_14 -->|main| update_vendor_matching_data_13
  notEmpty_14 -->|main | item 2| update_vendor_matching_data2_17
  notEmpty_14 -->|main | out 2| update_vendor_matching_data1_15
  notEmpty_14 -->|main | out 2 | item 2| update_vendor_matching_data3_18
  update_vendor_matching_data1_15 -->|main| Merge1_20
  update_vendor_matching_data2_17 -->|main| Merge_19
  update_vendor_matching_data3_18 -->|main| Merge1_20
  Merge_19 -->|main| Edit_Fields1_7
  Merge1_20 -->|main| Edit_Fields_16
```
