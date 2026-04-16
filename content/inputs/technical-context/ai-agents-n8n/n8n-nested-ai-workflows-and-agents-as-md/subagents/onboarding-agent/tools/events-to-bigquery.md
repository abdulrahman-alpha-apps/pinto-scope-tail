---
title: "Events to bigQuery"
publish: true
---

# Events to bigQuery

Source: `workflows/remote/dev_4/subagents/Onboarding agent/tools/Events to bigQuery.json`

Purpose:
Shared analytics sink for workflow event data.

Technical flow:
The workflow validates that an event payload is complete enough to log, reshapes it into the BigQuery schema, inserts it into BigQuery, and records an error object if validation fails.

Why it appears in multiple agents:
Several agents call this same workflow so all domain events land in a consistent analytics pipeline.

## Diagram

```mermaid
flowchart TD
  Validate_Event_1["Validate Event"]
  Format_Event_Data_2["Format Event Data"]
  Insert_to_BigQuery_3["Insert to BigQuery"]
  Log_Error_4["Log Error"]
  Sticky_Note_5["Sticky Note"]
  When_Executed_by_Another_Workflow_6["When Executed by Another Workflow"]
  Validate_Event_1 -->|main| Format_Event_Data_2
  Validate_Event_1 -->|main | out 2| Log_Error_4
  Format_Event_Data_2 -->|main| Insert_to_BigQuery_3
  When_Executed_by_Another_Workflow_6 -->|main| Validate_Event_1
```
