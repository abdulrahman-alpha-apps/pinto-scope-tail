---
title: "Extract Payment Info from CSV File abd"
publish: true
---

# Extract Payment Info from CSV File abd

Source: `workflows/remote/dev_4/subagents/Payment agent/tools/Extract Payment Info from CSV File abd.json`

Purpose:
Extracts payment instructions from a CSV upload.

Technical flow:
It fetches the CSV file, parses rows, runs an AI analysis step to interpret the payment information, uses code to normalize the result, and returns a cleaned payment payload to the payment agent.

## Diagram

```mermaid
flowchart TD
  Fetch_File_1["Fetch File"]
  Extract_From_CSV_2["Extract From CSV"]
  When_Executed_by_Another_Workflow_3["When Executed by Another Workflow"]
  Chat_Memory_Manager_4["Chat Memory Manager"]
  Postgres_Chat_Memory_5["Postgres Chat Memory"]
  Analyze_with_GPT_6["Analyze with GPT"]
  Code_7["Code"]
  Edit_Fields_8["Edit Fields"]
  Fetch_File_1 -->|main| Extract_From_CSV_2
  Extract_From_CSV_2 -->|main| Analyze_with_GPT_6
  When_Executed_by_Another_Workflow_3 -->|main| Fetch_File_1
  Chat_Memory_Manager_4 -->|main| Edit_Fields_8
  Postgres_Chat_Memory_5 -.->|ai_memory| Chat_Memory_Manager_4
  Analyze_with_GPT_6 -->|main| Code_7
  Code_7 -->|main| Chat_Memory_Manager_4
```
