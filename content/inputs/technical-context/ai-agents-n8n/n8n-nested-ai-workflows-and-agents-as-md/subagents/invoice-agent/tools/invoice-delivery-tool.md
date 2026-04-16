---
title: "Invoice Delivery Tool"
publish: true
---

# Invoice Delivery Tool

Source: `workflows/remote/dev_4/subagents/Invoice agent/tools/Invoice Delivery Tool.json`

Purpose:
Handles the delivery step after an invoice has been created.

Technical flow:
The workflow switches between delivery scenarios, validates whether phone and email data exist, updates missing customer contact details when possible, calls the invoice action/delivery endpoint, and returns either success data or structured error values.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  Switch_2["Switch"]
  Edit_Fields_3["Edit Fields"]
  If1_4["If1"]
  phoneIsNotEmpty_5["phoneIsNotEmpty"]
  Edit_Fields1_6["Edit Fields1"]
  Edit_Fields2_7["Edit Fields2"]
  Update_phoneNumber_8["Update phoneNumber"]
  Update_Email_9["Update Email"]
  Invoice_action_10["Invoice action"]
  Error_values_11["Error values"]
  Switch1_12["Switch1"]
  Edit_Fields3_13["Edit Fields3"]
  When_Executed_by_Another_Workflow_1 -->|main| Switch_2
  Switch_2 -->|main| phoneIsNotEmpty_5
  Switch_2 -->|main | out 2| If1_4
  Switch_2 -->|main | out 3| Invoice_action_10
  Switch_2 -->|main | out 4| Error_values_11
  phoneIsNotEmpty_5 -->|main| Update_phoneNumber_8
  phoneIsNotEmpty_5 -->|main | out 2| Edit_Fields_3
  If1_4 -->|main| Update_Email_9
  If1_4 -->|main | out 2| Edit_Fields1_6
  Update_phoneNumber_8 -->|main| Invoice_action_10
  Update_Email_9 -->|main| Invoice_action_10
  Update_Email_9 -->|main | out 2| Switch1_12
  Switch1_12 -->|main| Edit_Fields3_13
```
