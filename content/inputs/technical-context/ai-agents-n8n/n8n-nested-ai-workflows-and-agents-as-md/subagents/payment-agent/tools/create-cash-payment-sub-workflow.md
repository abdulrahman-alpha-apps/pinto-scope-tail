---
title: "Create Cash Payment Sub-Workflow"
publish: true
---

# Create Cash Payment Sub-Workflow

Source: `workflows/remote/dev_4/subagents/Payment agent/tools/Create Cash Payment Sub-Workflow.json`

Purpose:
Posts a cash payment transaction.

Technical flow:
The workflow builds the payment JSON payload in code, calls the payment API, inspects local state to confirm whether additional account lookup is required, and, if necessary, calls the Get Payment Account helper before returning the final payment result.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  Build_JSON_Payload_2["Build JSON Payload"]
  Make_Cash_Payment_API_Call_3["Make Cash Payment API Call"]
  Code_in_JavaScript_4["Code in JavaScript"]
  Get_Data_5["Get Data"]
  If_6["If"]
  Call__Get_Payment_Account__7["Call 'Get Payment Account'"]
  When_Executed_by_Another_Workflow_1 -->|main| Get_Data_5
  Build_JSON_Payload_2 -->|main| If_6
  Make_Cash_Payment_API_Call_3 -->|main| Code_in_JavaScript_4
  Get_Data_5 -->|main| Build_JSON_Payload_2
  If_6 -->|main| Make_Cash_Payment_API_Call_3
  If_6 -->|main | out 2| Call__Get_Payment_Account__7
  Call__Get_Payment_Account__7 -->|main| Get_Data_5
```
