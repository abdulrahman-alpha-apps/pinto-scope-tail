---
title: "Get Payment Account"
publish: true
---

# Get Payment Account

Source: `workflows/remote/dev_4/subagents/Payment agent/tools/Get Payment Account.json`

Purpose:
Selects the right payment account for a payment operation.

Technical flow:
The workflow fetches available payment accounts, summarizes them for an AI model, uses an AI agent and structured parser to pick the best account, runs JavaScript validation logic, and stores the selected account in data tables for downstream payment posting.

## Diagram

```mermaid
flowchart TD
  Start_1["Start"]
  Postgres_Chat_Memory_2["Postgres Chat Memory"]
  Payment_accounts_3["Payment accounts"]
  Message_a_model_4["Message a model"]
  Edit_Fields_5["Edit Fields"]
  0050301_6["0050301"]
  0050302_7["0050302"]
  AI_Agent_8["AI Agent"]
  OpenAI_Chat_Model_9["OpenAI Chat Model"]
  Code_in_JavaScript1_10["Code in JavaScript1"]
  If_11["If"]
  Edit_Fields1_12["Edit Fields1"]
  Upsert_row_s_4_13["Upsert row(s)4"]
  Structured_Output_Parser_14["Structured Output Parser"]
  Start_1 -->|main| Payment_accounts_3
  Payment_accounts_3 -->|main| Code_in_JavaScript1_10
  Payment_accounts_3 -->|main | out 2| 0050301_6
  AI_Agent_8 -->|main| Upsert_row_s_4_13
  OpenAI_Chat_Model_9 -.->|ai_languageModel| AI_Agent_8
  OpenAI_Chat_Model_9 -.->|ai_languageModel | item 2| Structured_Output_Parser_14
  Code_in_JavaScript1_10 -->|main| If_11
  If_11 -->|main| Edit_Fields1_12
  If_11 -->|main | out 2| AI_Agent_8
  Edit_Fields1_12 -->|main| Upsert_row_s_4_13
  Upsert_row_s_4_13 -->|main| Edit_Fields_5
  Structured_Output_Parser_14 -.->|ai_outputParser| AI_Agent_8
```
