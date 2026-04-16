---
title: "Timeout Notifier - State Check"
publish: true
---

# Timeout Notifier - State Check

Source: `workflows/remote/dev_4/subagents/Invoice agent/tools/Timeout Notifier - State Check.json`

Purpose:
Sends a reminder if an invoice conversation becomes stuck.

Technical flow:
The workflow waits 15 seconds, checks the current timeout/completion state from a data table, and either sends a timeout message or exits without action if the invoice thread has already completed.

## Diagram

```mermaid
flowchart TD
  Workflow_Input_1["Workflow Input"]
  Wait_15_Seconds_2["Wait 15 Seconds"]
  Should_Send_Message__3["Should Send Message?"]
  Send_Timeout_Message_4["Send Timeout Message"]
  Workflow_Already_Completed_5["Workflow Already Completed"]
  Get_row_s__6["Get row(s)"]
  Workflow_Input_1 -->|main| Wait_15_Seconds_2
  Wait_15_Seconds_2 -->|main| Get_row_s__6
  Should_Send_Message__3 -->|main| Send_Timeout_Message_4
  Should_Send_Message__3 -->|main | out 2| Workflow_Already_Completed_5
  Get_row_s__6 -->|main| Should_Send_Message__3
```
