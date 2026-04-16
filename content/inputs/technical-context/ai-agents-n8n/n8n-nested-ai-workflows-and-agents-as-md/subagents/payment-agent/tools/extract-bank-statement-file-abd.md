---
title: "Extract Bank Statement File abd"
publish: true
---

# Extract Bank Statement File abd

Source: `workflows/remote/dev_4/subagents/Payment agent/tools/Extract Bank Statement File abd.json`

Purpose:
Extracts structured bank-statement data from uploaded files.

Technical flow:
The workflow fetches the file, detects whether it should use generic file extraction or CSV-specific extraction, and then uses AI analysis plus chat memory to turn the content into a normalized bank-statement structure that the payment agent can reason about.

## Diagram

```mermaid
flowchart TD
  Fetch_File_1["Fetch File"]
  Extract_From_File_2["Extract From File"]
  When_Executed_by_Another_Workflow_3["When Executed by Another Workflow"]
  Chat_Memory_Manager_4["Chat Memory Manager"]
  Postgres_Chat_Memory_5["Postgres Chat Memory"]
  Edit_Fields_6["Edit Fields"]
  If_7["If"]
  Extract_From_CSV_8["Extract From CSV"]
  Analyze_with_GPT1_9["Analyze with GPT1"]
  Analyze_with_GPT_10["Analyze with GPT"]
  Chat_Memory_Manager1_11["Chat Memory Manager1"]
  Postgres_Chat_Memory1_12["Postgres Chat Memory1"]
  Edit_Fields1_13["Edit Fields1"]
  Fetch_File_1 -->|main| If_7
  Extract_From_File_2 -->|main| Analyze_with_GPT_10
  When_Executed_by_Another_Workflow_3 -->|main| Fetch_File_1
  Chat_Memory_Manager_4 -->|main| Edit_Fields_6
  Postgres_Chat_Memory_5 -.->|ai_memory| Chat_Memory_Manager_4
  If_7 -->|main| Extract_From_File_2
  If_7 -->|main | out 2| Extract_From_CSV_8
  Extract_From_CSV_8 -->|main| Analyze_with_GPT1_9
  Analyze_with_GPT1_9 -->|main| Chat_Memory_Manager1_11
  Analyze_with_GPT_10 -->|main| Chat_Memory_Manager_4
  Chat_Memory_Manager1_11 -->|main| Edit_Fields1_13
  Postgres_Chat_Memory1_12 -.->|ai_memory| Chat_Memory_Manager1_11
```
