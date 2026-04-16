---
title: "Create items"
publish: true
---

# Create items

Source: `workflows/remote/dev_4/subagents/Invoice agent/tools/Create items.json`

Purpose:
Creates one or more invoice items/products needed by the invoice workflow.

Technical flow:
The workflow transforms incoming item data, loops over each item, calls the item-creation API, aggregates the created item results, and stores the final item mapping for later invoice submission.

## Diagram

```mermaid
flowchart TD
  Start_1["Start"]
  Edit_Fields1_2["Edit Fields1"]
  create_items_3["create items"]
  Edit_Fields_4["Edit Fields"]
  Loop_Over_Items1_5["Loop Over Items1"]
  Edit_Fields3_6["Edit Fields3"]
  Aggregate_7["Aggregate"]
  Upsert_row_s__8["Upsert row(s)"]
  Edit_Fields2_9["Edit Fields2"]
  Start_1 -->|main| Edit_Fields_4
  Edit_Fields_4 -->|main| create_items_3
  create_items_3 -->|main| Loop_Over_Items1_5
  create_items_3 -->|main | out 2| Edit_Fields2_9
  Loop_Over_Items1_5 -->|main| Aggregate_7
  Loop_Over_Items1_5 -->|main | out 2| Edit_Fields3_6
  Edit_Fields3_6 -->|main| Loop_Over_Items1_5
  Aggregate_7 -->|main| Upsert_row_s__8
  Upsert_row_s__8 -->|main| Edit_Fields1_2
```
