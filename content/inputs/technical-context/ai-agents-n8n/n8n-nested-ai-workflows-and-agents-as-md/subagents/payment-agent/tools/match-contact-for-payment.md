---
title: "Match contact for payment"
publish: true
---

# Match contact for payment

Source: `workflows/remote/dev_4/subagents/Payment agent/tools/Match contact for payment.json`

Purpose:
Finds the most likely contact for a payment thread.

Technical flow:
An AI agent queries a contact-matching HTTP tool, parses the result, checks whether the match is non-empty, sends a clarification message when required, and stores the chosen contact in a data table and memory for reuse by later payment steps.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  AI_Agent_2["AI Agent"]
  OpenAI_Chat_Model_3["OpenAI Chat Model"]
  Structured_Output_Parser_4["Structured Output Parser"]
  Match_contact_5["Match contact"]
  Send_message_on_whatsapp_6["Send message on whatsapp"]
  notEmpty_7["notEmpty"]
  Edit_Fields_8["Edit Fields"]
  Edit_Fields2_9["Edit Fields2"]
  Insert_matched_contact_10["Insert matched contact"]
  Simple_Memory_11["Simple Memory"]
  When_Executed_by_Another_Workflow_1 -->|main| AI_Agent_2
  When_Executed_by_Another_Workflow_1 -->|main | item 2| Send_message_on_whatsapp_6
  OpenAI_Chat_Model_3 -.->|ai_languageModel| AI_Agent_2
  Structured_Output_Parser_4 -.->|ai_outputParser| AI_Agent_2
  AI_Agent_2 -->|main| notEmpty_7
  Match_contact_5 -.->|ai_tool| AI_Agent_2
  notEmpty_7 -->|main| Insert_matched_contact_10
  notEmpty_7 -->|main | out 2| Edit_Fields_8
  Insert_matched_contact_10 -->|main| Edit_Fields2_9
  Simple_Memory_11 -.->|ai_memory| AI_Agent_2
```
