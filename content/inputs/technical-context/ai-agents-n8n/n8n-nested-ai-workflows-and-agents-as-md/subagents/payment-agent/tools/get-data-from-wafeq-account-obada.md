---
title: "Get data from wafeq account obada"
publish: true
---

# Get data from wafeq account obada

Source: `workflows/remote/dev_4/subagents/Payment agent/tools/Get data from wafeq account obada.json`

Purpose:
Retrieves account and tax reference data from Wafeq.

Technical flow:
The workflow calls external APIs for accounts and tax rates, filters the returned structures in code, merges the datasets, aggregates them into a usable result, and stores the transformed payload in memory/state for later payment decisions.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  Filter_Taxes1_2["Filter Taxes1"]
  Filter_COA1_3["Filter COA1"]
  Get_Tax_Rates1_4["Get Tax Rates1"]
  Merge1_5["Merge1"]
  Get_accounts1_6["Get accounts1"]
  Aggregate_7["Aggregate"]
  Chat_Memory_Manager_8["Chat Memory Manager"]
  Postgres_Chat_Memory_9["Postgres Chat Memory"]
  Edit_Fields_10["Edit Fields"]
  0050201_11["0050201"]
  0050202_12["0050202"]
  When_Executed_by_Another_Workflow_1 -->|main| Get_Tax_Rates1_4
  When_Executed_by_Another_Workflow_1 -->|main | item 2| Get_accounts1_6
  Filter_Taxes1_2 -->|main| Merge1_5
  Filter_COA1_3 -->|main| Merge1_5
  Get_Tax_Rates1_4 -->|main| Filter_Taxes1_2
  Get_Tax_Rates1_4 -->|main | out 2| 0050202_12
  Get_accounts1_6 -->|main| Filter_COA1_3
  Get_accounts1_6 -->|main | out 2| 0050201_11
  Merge1_5 -->|main| Aggregate_7
  Postgres_Chat_Memory_9 -.->|ai_memory| Chat_Memory_Manager_8
  Aggregate_7 -->|main| Chat_Memory_Manager_8
  Chat_Memory_Manager_8 -->|main| Edit_Fields_10
```
