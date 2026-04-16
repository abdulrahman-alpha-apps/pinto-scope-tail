---
title: "Match items"
publish: true
---

# Match items

Source: `workflows/remote/dev_4/subagents/Invoice agent/tools/Match items.json`

Purpose:
Matches invoice line descriptions to existing item records.

Technical flow:
The workflow normalizes item names, uses an AI agent plus a structured parser to search for likely catalog matches, stores the selected mapping in data tables, and returns a clean item-resolution object to the invoice agent.

## Diagram

```mermaid
flowchart TD
  Start_1["Start"]
  Parser_for_items_2_2["Parser for items 2"]
  AI_Agent___Item_Matcher_3["AI Agent - Item Matcher"]
  Match_Items_Tool_4["Match Items Tool"]
  OpenAI_Chat_Model1_5["OpenAI Chat Model1"]
  Structured_Output_Parser1_6["Structured Output Parser1"]
  Simple_Memory1_7["Simple Memory1"]
  names_only_8["names only"]
  Upsert_row_s__9["Upsert row(s)"]
  Edit_Fields_10["Edit Fields"]
  Start_1 -->|main| Parser_for_items_2_2
  Parser_for_items_2_2 -->|main| names_only_8
  Match_Items_Tool_4 -.->|ai_tool| AI_Agent___Item_Matcher_3
  OpenAI_Chat_Model1_5 -.->|ai_languageModel| AI_Agent___Item_Matcher_3
  OpenAI_Chat_Model1_5 -.->|ai_languageModel | item 2| Structured_Output_Parser1_6
  Structured_Output_Parser1_6 -.->|ai_outputParser| AI_Agent___Item_Matcher_3
  Simple_Memory1_7 -.->|ai_memory| AI_Agent___Item_Matcher_3
  names_only_8 -->|main| AI_Agent___Item_Matcher_3
  AI_Agent___Item_Matcher_3 -->|main| Upsert_row_s__9
  Upsert_row_s__9 -->|main| Edit_Fields_10
```
