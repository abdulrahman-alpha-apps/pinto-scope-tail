---
title: "selecting Account Expense"
publish: true
---

# selecting Account Expense

Source: `workflows/remote/dev_4/subagents/Expense agent V3/tools/selecting Account Expense.json`

Purpose:
Chooses the correct expense chart-of-account entry.

Technical flow:
The workflow fetches available accounts, filters the list in code, and then uses an AI agent with a structured output parser to map the user’s description or OCR context to the most appropriate account. It stores the chosen result in data tables and memory so the expense conversation can continue without asking the same question again.

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
  Upsert_row_s__12["Upsert row(s)"]
  Merge_13["Merge"]
  When_Executed_by_Another_Workflow_1 -->|main| replace_with_start_7
  Get_accounts1_2 -->|main| Filter_COA_4
  OpenAI_Chat_Model_3 -.->|ai_languageModel| AI_Agent_5
  Filter_COA_4 -->|main| AI_Agent_5
  Structured_Output_Parser_6 -.->|ai_outputParser| AI_Agent_5
  replace_with_start_7 -->|main| Get_accounts1_2
  Chat_Memory_Manager_8 -->|main| Upsert_row_s_4_11
  Postgres_Chat_Memory_9 -.->|ai_memory| Chat_Memory_Manager_8
  AI_Agent_5 -->|main| Chat_Memory_Manager_8
  AI_Agent_5 -->|main | item 2| Upsert_row_s__12
  Upsert_row_s_4_11 -->|main| Merge_13
  Upsert_row_s__12 -->|main| Merge_13
  Merge_13 -->|main| Edit_Fields1_10
```
