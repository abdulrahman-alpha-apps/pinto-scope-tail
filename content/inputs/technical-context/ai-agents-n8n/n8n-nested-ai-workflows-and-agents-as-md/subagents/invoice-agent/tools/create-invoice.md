---
title: "Create invoice"
publish: true
---

# Create invoice

Source: `workflows/remote/dev_4/subagents/Invoice agent/tools/Create invoice.json`

Purpose:
Builds and submits the final invoice payload.

Technical flow:
The workflow prepares invoice line items and headers in JavaScript, posts the invoice to the target system, checks whether VAT/TRN should be skipped or validated, calls the VAT health-check workflow and the account-selection helper when needed, merges the resulting data, and returns a normalized invoice outcome back to the agent.

## Diagram

```mermaid
flowchart TD
  Start_1["Start"]
  Code_2["Code"]
  HTTP_Request_3["HTTP Request"]
  Code_in_JavaScript_4["Code in JavaScript"]
  TRN_is_skipped_5["TRN is skipped"]
  Call__check_VAT_TRN_health__6["Call 'check VAT TRN health'"]
  Call__selecting_Account_Invoice__7["Call 'selecting Account Invoice'"]
  Merge_8["Merge"]
  Code_in_JavaScript1_9["Code in JavaScript1"]
  Edit_Fields_10["Edit Fields"]
  Edit_Fields1_11["Edit Fields1"]
  Send_message_on_whatsapp_12["Send message on whatsapp"]
  Start_1 -->|main| Code_2
  Start_1 -->|main | item 2| Send_message_on_whatsapp_12
  Code_2 -->|main| Call__selecting_Account_Invoice__7
  Code_in_JavaScript_4 -->|main| HTTP_Request_3
  TRN_is_skipped_5 -->|main| Code_in_JavaScript_4
  TRN_is_skipped_5 -->|main | out 2| HTTP_Request_3
  Call__selecting_Account_Invoice__7 -->|main| Code_in_JavaScript1_9
  Code_in_JavaScript1_9 -->|main| TRN_is_skipped_5
  HTTP_Request_3 -->|main| Edit_Fields1_11
  HTTP_Request_3 -->|main | out 2| Edit_Fields_10
```
