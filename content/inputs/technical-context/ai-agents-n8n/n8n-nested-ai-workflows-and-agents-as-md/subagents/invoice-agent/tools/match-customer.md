---
title: "Match customer"
publish: true
---

# Match customer

Source: `workflows/remote/dev_4/subagents/Invoice agent/tools/Match customer.json`

Purpose:
Finds the best existing customer match for an invoice.

Technical flow:
An AI agent receives the user/customer context, queries an HTTP matching tool, parses the result into a structured answer, writes the selected match into data tables, and optionally sends a WhatsApp clarification message when the match is weak or ambiguous.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  AI_Agent_2["AI Agent"]
  OpenAI_Chat_Model_3["OpenAI Chat Model"]
  Structured_Output_Parser_4["Structured Output Parser"]
  Edit_Fields1_5["Edit Fields1"]
  Match_contact_6["Match contact"]
  HTTP_Request_7["HTTP Request"]
  Upsert_row_s__8["Upsert row(s)"]
  Send_message_on_whatsapp_9["Send message on whatsapp"]
  When_Executed_by_Another_Workflow_1 -->|main| AI_Agent_2
  When_Executed_by_Another_Workflow_1 -->|main | item 2| Send_message_on_whatsapp_9
  OpenAI_Chat_Model_3 -.->|ai_languageModel| AI_Agent_2
  Structured_Output_Parser_4 -.->|ai_outputParser| AI_Agent_2
  AI_Agent_2 -->|main| Upsert_row_s__8
  Match_contact_6 -.->|ai_tool| AI_Agent_2
  Upsert_row_s__8 -->|main| Edit_Fields1_5
```
