---
title: "Skip TRN for wafeq"
publish: true
---

# Skip TRN for wafeq

Source: `workflows/remote/dev_4/subagents/skip TRN for wafeq/skip TRN for wafeq.json`

Purpose:
Records the decision to skip TRN/VAT creation for a Wafeq organization.

Technical flow:
The workflow marks the VAT choice as a skip/no-creation case, appends the decision to Google Sheets, and writes the same state into internal data tables so the router and health-check logic know that the omission was intentional.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  no_creation_2["no creation"]
  VAT_3["VAT"]
  Append_or_update_row_in_sheet_4["Append or update row in sheet"]
  Upsert_row_s__5["Upsert row(s)"]
  Limit_6["Limit"]
  new_table_7["new table"]
  When_Executed_by_Another_Workflow_1 -->|main| Upsert_row_s__5
  When_Executed_by_Another_Workflow_1 -->|main | item 2| new_table_7
  Upsert_row_s__5 -->|main| Limit_6
  Limit_6 -->|main| VAT_3
  new_table_7 -->|main| Limit_6
```
