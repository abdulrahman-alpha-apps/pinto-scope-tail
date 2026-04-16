---
title: "Check VAT TRN health"
publish: true
---

# Check VAT TRN health

Source: `workflows/remote/dev_4/subagents/check VAT TRN health/check VAT TRN health.json`

Purpose:
Shared validator and reconciler for VAT/TRN setup across Zoho and Wafeq.

Technical flow:
This workflow receives an organization context, branches between Zoho and Wafeq, fetches remote tax-rate or VAT data, splits and filters the returned structures, aggregates normalized results, and compares them against internal table state. It then upserts the latest health information, raises task-creation calls for mismatches, and records whether the TRN was updated successfully.

Why it matters:
This is the control workflow that prevents the system from assuming a TRN/VAT setup is healthy when the external accounting platform and internal state disagree.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  zoho_2["zoho"]
  wafeq_3["wafeq"]
  zoho_rate_4["zoho rate"]
  Split_Out_5["Split Out"]
  Filter_6["Filter"]
  Filter1_7["Filter1"]
  Aggregate_8["Aggregate"]
  Edit_Fields_9["Edit Fields"]
  Edit_Fields1_10["Edit Fields1"]
  Code_in_JavaScript_11["Code in JavaScript"]
  Split_Out1_12["Split Out1"]
  wafek_13["wafek"]
  Code_in_JavaScript1_14["Code in JavaScript1"]
  is_it_zoho_15["is it zoho"]
  Filter2_16["Filter2"]
  If_17["If"]
  If1_18["If1"]
  Update_row_s_1_19["Update row(s)1"]
  Upsert_row_s__20["Upsert row(s)"]
  Filter3_21["Filter3"]
  Merge_22["Merge"]
  Aggregate1_23["Aggregate1"]
  Edit_Fields2_24["Edit Fields2"]
  Merge1_25["Merge1"]
  If2_26["If2"]
  Aggregate2_27["Aggregate2"]
  If3_28["If3"]
  Edit_Fields3_29["Edit Fields3"]
  Upsert_row_s_1_30["Upsert row(s)1"]
  Sticky_Note_31["Sticky Note"]
  Sticky_Note1_32["Sticky Note1"]
  Sticky_Note2_33["Sticky Note2"]
  Sticky_Note3_34["Sticky Note3"]
  Edit_Fields4_35["Edit Fields4"]
  click_36["click"]
  zoho1_37["zoho1"]
  zoho_rate1_38["zoho rate1"]
  Upsert_row_s_3_39["Upsert row(s)3"]
  Sticky_Note4_40["Sticky Note4"]
  creating_a_task_41["creating a task"]
  Edit_Fields5_42["Edit Fields5"]
  creating_a_task1_43["creating a task1"]
  If_row_exists_44["If row exists"]
  If_row_exists1_45["If row exists1"]
  Upsert_row_s_2_46["Upsert row(s)2"]
  trn_updated_47["trn updated"]
  trn_updated1_48["trn updated1"]
  Upsert_row_s_4_49["Upsert row(s)4"]
  When_Executed_by_Another_Workflow_1 -->|main| is_it_zoho_15
  zoho_2 -->|main| zoho_rate_4
  wafeq_3 -->|main| wafek_13
  zoho_rate_4 -->|main| Split_Out_5
  Split_Out_5 -->|main| Code_in_JavaScript_11
  Split_Out_5 -->|main | item 2| Filter3_21
  Aggregate_8 -->|main| Edit_Fields_9
  Filter_6 -->|main| Edit_Fields1_10
  wafek_13 -->|main| Split_Out1_12
  Split_Out1_12 -->|main| Code_in_JavaScript1_14
  Split_Out1_12 -->|main | item 2| Filter2_16
  is_it_zoho_15 -->|main| zoho_rate_4
  is_it_zoho_15 -->|main | item 2| trn_updated1_48
  is_it_zoho_15 -->|main | out 2| wafek_13
  is_it_zoho_15 -->|main | out 2 | item 2| trn_updated_47
  Filter2_16 -->|main| Merge1_25
  If_17 -->|main| Merge_22
  If_17 -->|main | out 2| creating_a_task_41
  If1_18 -->|main| Merge1_25
  If1_18 -->|main | out 2| creating_a_task_41
  Code_in_JavaScript1_14 -->|main| If1_18
  Code_in_JavaScript_11 -->|main| If_17
  Filter3_21 -->|main| Merge_22
  Merge_22 -->|main| Aggregate1_23
  Aggregate1_23 -->|main| If2_26
  Edit_Fields2_24 -->|main| Upsert_row_s__20
  If2_26 -->|main| Edit_Fields2_24
  Merge1_25 -->|main| Aggregate2_27
  Aggregate2_27 -->|main| If3_28
  If3_28 -->|main | out 2| Edit_Fields3_29
  Edit_Fields3_29 -->|main| Upsert_row_s_1_30
  Upsert_row_s_1_30 -->|main| Edit_Fields4_35
  Upsert_row_s__20 -->|main| Edit_Fields4_35
  click_36 -->|main| zoho1_37
  zoho1_37 -->|main| zoho_rate1_38
  creating_a_task_41 -->|main| Edit_Fields5_42
  trn_updated_47 -->|main| Upsert_row_s_2_46
  trn_updated1_48 -->|main| Upsert_row_s_4_49
```
