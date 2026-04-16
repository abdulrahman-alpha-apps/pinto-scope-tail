---
title: "selecting Account Invoice"
publish: true
---

# selecting Account Invoice

Source: `workflows/remote/dev_4/subagents/Invoice agent/tools/selecting Account Invoice.json`

Purpose:
Selects the correct revenue/account mapping for invoice posting.

Technical flow:
The workflow retrieves account choices, filters them in code, uses an AI agent and structured parser to choose the most suitable account, and stores the decision in chat memory and data tables for the rest of the invoice flow.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  Get_accounts1_2["Get accounts1"]
  OpenAI_Chat_Model_3["OpenAI Chat Model"]
  Filter_COA_4["Filter COA"]
  AI_Agent_5["AI Agent"]
  Structured_Output_Parser_6["Structured Output Parser"]
  replace_with_start_7["replace with start"]
  Chat_Memory_Manager_8["Chat Memory Manager"]
  Postgres_Chat_Memory_9["Postgres Chat Memory"]
  Edit_Fields1_10["Edit Fields1"]
  Upsert_row_s_4_11["Upsert row(s)4"]
  When_Executed_by_Another_Workflow_1 -->|main| replace_with_start_7
  Get_accounts1_2 -->|main| Filter_COA_4
  OpenAI_Chat_Model_3 -.->|ai_languageModel| AI_Agent_5
  OpenAI_Chat_Model_3 -.->|ai_languageModel | item 2| Structured_Output_Parser_6
  Filter_COA_4 -->|main| AI_Agent_5
  Structured_Output_Parser_6 -.->|ai_outputParser| AI_Agent_5
  replace_with_start_7 -->|main| Get_accounts1_2
  Chat_Memory_Manager_8 -->|main| Upsert_row_s_4_11
  Postgres_Chat_Memory_9 -.->|ai_memory| Chat_Memory_Manager_8
  AI_Agent_5 -->|main| Chat_Memory_Manager_8
  Upsert_row_s_4_11 -->|main| Edit_Fields1_10
```
