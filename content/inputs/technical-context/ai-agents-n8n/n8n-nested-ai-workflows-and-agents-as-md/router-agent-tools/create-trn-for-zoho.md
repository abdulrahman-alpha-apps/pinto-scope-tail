---
title: "Create TRN for zoho"
publish: true
---

# Create TRN for zoho

Source: `workflows/remote/dev_4/subagents/Create TRN for zoho/Create TRN for zoho.json`

Purpose:
Creates or prepares a VAT/TRN record for Zoho-connected organizations.

Technical flow:
This sub-workflow is triggered by another workflow. It checks whether the organization already has a TRN, builds the VAT payload, optionally calls an external HTTP endpoint to create the VAT record, writes the result into internal tables, and then calls the shared VAT health-check workflow to validate the created state.

Why it exists:
The router uses this workflow when the conversation reaches the point where a Zoho customer needs a TRN/VAT configuration instead of just skipping it.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  Create_Vat_2["Create Vat"]
  no_creation_3["no creation"]
  VAT_4["VAT"]
  new_table_5["new table"]
  Call__check_VAT_TRN_health__6["Call 'check VAT TRN health'"]
  Edit_Fields_7["Edit Fields"]
  Edit_Fields1_8["Edit Fields1"]
  If_already_have_org_trn_9["If already have org trn"]
  When_Executed_by_Another_Workflow_1 -->|main| Create_Vat_2
  Create_Vat_2 -->|main| new_table_5
  Create_Vat_2 -->|main | out 2| If_already_have_org_trn_9
  new_table_5 -->|main| VAT_4
  new_table_5 -->|main | item 2| Call__check_VAT_TRN_health__6
  If_already_have_org_trn_9 -->|main| Edit_Fields_7
  If_already_have_org_trn_9 -->|main | out 2| Edit_Fields1_8
```
