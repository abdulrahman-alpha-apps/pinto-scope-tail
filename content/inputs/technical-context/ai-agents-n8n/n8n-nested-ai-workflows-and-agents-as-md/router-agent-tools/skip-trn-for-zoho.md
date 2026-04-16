---
title: "Skip TRN for zoho"
publish: true
---

# Skip TRN for zoho

Source: `workflows/remote/dev_4/subagents/skip TRN for zoho/skip TRN for zoho.json`

Purpose:
Records the decision to skip TRN/VAT creation for a Zoho organization.

Technical flow:
It writes a no-creation VAT decision into Google Sheets and internal tables, making the skip state visible to the router and to later VAT health-check logic.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  no_creation_2["no creation"]
  VAT_3["VAT"]
  Append_or_update_row_in_sheet_4["Append or update row in sheet"]
  Upsert_row_s__5["Upsert row(s)"]
  new_table1_6["new table1"]
  When_Executed_by_Another_Workflow_1 -->|main| Append_or_update_row_in_sheet_4
  When_Executed_by_Another_Workflow_1 -->|main | item 2| new_table1_6
  Append_or_update_row_in_sheet_4 -->|main| Upsert_row_s__5
  Upsert_row_s__5 -->|main| VAT_3
  new_table1_6 -->|main| VAT_3
```
