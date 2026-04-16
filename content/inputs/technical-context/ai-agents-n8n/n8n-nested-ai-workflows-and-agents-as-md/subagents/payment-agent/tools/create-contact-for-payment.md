---
title: "Create contact for payment"
publish: true
---

# Create contact for payment

Source: `workflows/remote/dev_4/subagents/Payment agent/tools/Create contact for payment.json`

Purpose:
Creates a payment contact when no suitable existing contact can be matched.

Technical flow:
It checks for existing matches first, loops through candidates, waits for controlled retries or pacing, creates the contact when needed, sends user-facing updates, and stores the new or matched contact identity for the payment flow.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  Edit_Fields1_2["Edit Fields1"]
  Send_message_on_whatsapp_3["Send message on whatsapp"]
  match_as_check_4["match as check"]
  Code_5["Code"]
  Wait1_6["Wait1"]
  Loop_Over_Items_7["Loop Over Items"]
  If_8["If"]
  0050502_9["0050502"]
  0050503_10["0050503"]
  create_11["create"]
  Edit_Fields_12["Edit Fields"]
  Insert_matched_contact_13["Insert matched contact"]
  When_Executed_by_Another_Workflow_1 -->|main| Send_message_on_whatsapp_3
  When_Executed_by_Another_Workflow_1 -->|main | item 2| create_11
  match_as_check_4 -->|main| Code_5
  match_as_check_4 -->|main | out 2| 0050502_9
  Code_5 -->|main| If_8
  Code_5 -->|main | out 2| 0050503_10
  Wait1_6 -->|main| Loop_Over_Items_7
  Loop_Over_Items_7 -->|main | out 2| match_as_check_4
  If_8 -->|main| Insert_matched_contact_13
  If_8 -->|main | out 2| Wait1_6
  create_11 -->|main| Loop_Over_Items_7
  create_11 -->|main | out 2| Edit_Fields_12
  Insert_matched_contact_13 -->|main| Edit_Fields1_2
```
