---
title: "Router agent"
publish: true
---

# Router agent

Source: `workflows/remote/dev_4/root/Router agent.json`

Purpose:
This is the system entry point. It receives the user interaction, normalizes the incoming payload, decides whether the message is onboarding, expense, invoice, payment, or reporting work, and then forwards the thread to the correct specialist workflow.

Technical flow:
The workflow starts from a webhook and also contains test/manual entry nodes. It enriches the request by reading thread state from Postgres/Data Tables, detects whether the payload is text, audio, CSV, PDF, expense data, or bank-statement data, and uses AI Agent nodes plus switch/if routing to choose the next action. For text conversations it routes to one of the domain agents. For attachments it pre-processes the file, stores intermediate extracted data, and only then routes the thread.

Subagents:
- Onboarding agent: handles onboarding and connection/setup flows.
- Expense agent: handles expense capture, OCR, vendor/account selection, and expense posting.
- Payment agent: handles payment extraction, contact/account selection, and payment posting.
- Invoice agent: handles invoice creation, customer/item resolution, and invoice delivery.
- Report agent: handles reporting and report-generation requests.

Router tools:
- Create TRN for zoho: creates or prepares VAT/TRN setup for Zoho organizations.
- Skip TRN for Zoho: records an intentional skip of TRN/VAT creation for Zoho.
- Skip TRN for Wafeq: records an intentional skip of TRN/VAT creation for Wafeq.
- Check VAT TRN health: validates whether external VAT/TRN state matches the internal system state.

TRN/VAT handling:
The router also owns a VAT/TRN branch. It can call the Zoho and Wafeq TRN helper agents, call the TRN health-check workflow, and decide whether the user is creating, skipping, or validating a tax registration number.

Outputs and side effects:
It sends WhatsApp replies, updates thread/router status, writes parsed file data into data tables, and launches the downstream agent workflows using Execute Workflow or Tool Workflow nodes.

## Diagram

```mermaid
flowchart TD
  Webhook_1["Webhook"]
  Onboarding_agent_2["Onboarding agent"]
  Invoice_agent_3["Invoice agent"]
  Payment_agent_4["Payment agent"]
  Report_agent_5["Report agent"]
  Expense_agent_6["Expense agent"]
  Switch_7["Switch"]
  If1_8["If1"]
  AI_Agent_9["AI Agent"]
  Simple_Memory1_10["Simple Memory1"]
  OpenAI_Chat_Model1_11["OpenAI Chat Model1"]
  Send_message_on_whatsapp_12["Send message on whatsapp"]
  Code1_13["Code1"]
  If2_14["If2"]
  Switch1_15["Switch1"]
  Send_message_on_whatsapp1_16["Send message on whatsapp1"]
  Get_state_17["Get state"]
  Reset_Number_18["Reset Number"]
  Sticky_Note_19["Sticky Note"]
  Create_TRN_for_zoho_20["Create TRN for zoho"]
  Skip_TRN_for_Zoho_21["Skip TRN for Zoho"]
  is_it_zoho_22["is it zoho"]
  OpenAI_Chat_Model3_23["OpenAI Chat Model3"]
  Zoho_TRN_24["Zoho TRN"]
  is_user_skipping_TRN_25["is user skipping TRN"]
  Org_ID_26["Org ID"]
  Simple_Memory_27["Simple Memory"]
  Send_message_on_whatsapp2_28["Send message on whatsapp2"]
  wafeq_TRN1_29["wafeq TRN1"]
  Skip_TRN_for_Wafeq_30["Skip TRN for Wafeq"]
  No_Operation__do_nothing_31["No Operation, do nothing"]
  TRN_32["TRN"]
  0010000_33["0010000"]
  0050000_34["0050000"]
  0030000_35["0030000"]
  0020000_36["0020000"]
  0040000_37["0040000"]
  Get_row_s__38["Get row(s)"]
  Limit_39["Limit"]
  Call__check_VAT_TRN_health__40["Call 'check VAT TRN health'"]
  Datadog_HTTP_Request_41["Datadog HTTP Request"]
  When_clicking__Execute_workflow__42["When clicking ‘Execute workflow’"]
  HTTP_Request_test_name_43["HTTP Request test name"]
  creating_a_task1_44["creating a task1"]
  Limit1_45["Limit1"]
  zoho_46["zoho"]
  wafeq_47["wafeq"]
  Edit_Fields_48["Edit Fields"]
  Call__check_VAT_TRN_health_2_49["Call 'check VAT TRN health'2"]
  Check_VAT_TRN_health_50["Check VAT TRN health"]
  Sticky_Note1_51["Sticky Note1"]
  Sticky_Note2_52["Sticky Note2"]
  Edit_Fields4_53["Edit Fields4"]
  Transcribe_a_recording_54["Transcribe a recording"]
  Is_it_Audio_55["Is it Audio"]
  Received_Values_56["Received Values"]
  Analyze_document_57["Analyze document"]
  is_the_file_csv_58["is the file csv"]
  is_the_file_pdf_59["is the file pdf"]
  Parsing_output_60["Parsing output"]
  Upsert_expense_data_61["Upsert expense data"]
  expense_or_bankStatement_62["expense or bankStatement"]
  Upsert_bankStatement_data_63["Upsert bankStatement data"]
  Extract_from_File_64["Extract from File"]
  HTTP_Request_65["HTTP Request"]
  Message_a_model_66["Message a model"]
  Upsert_bankStatement_data1_67["Upsert bankStatement data1"]
  Switch3_68["Switch3"]
  No_Operation__do_nothing1_69["No Operation, do nothing1"]
  Limit2_70["Limit2"]
  Message_injection_71["Message injection"]
  Message_update_72["Message update"]
  HTTP_Request1_73["HTTP Request1"]
  Get_accounts1_74["Get accounts1"]
  If_75["If"]
  Upsert_expense_data2_76["Upsert expense data2"]
  Get_data_to_check_if_its_first_time_77["Get data to check if its first time"]
  Update_thread_status_78["Update thread status"]
  Webhook_1 -->|main| Received_Values_56
  Onboarding_agent_2 -->|main | out 2| 0010000_33
  Switch_7 -->|main| AI_Agent_9
  Switch_7 -->|main | out 2| Expense_agent_6
  Switch_7 -->|main | out 3| Payment_agent_4
  Switch_7 -->|main | out 4| Invoice_agent_3
  Switch_7 -->|main | out 5| Report_agent_5
  If1_8 -->|main| Switch_7
  If1_8 -->|main | out 2| AI_Agent_9
  Simple_Memory1_10 -.->|ai_memory| AI_Agent_9
  OpenAI_Chat_Model1_11 -.->|ai_languageModel| AI_Agent_9
  AI_Agent_9 -->|main| Code1_13
  Code1_13 -->|main| If2_14
  If2_14 -->|main| Send_message_on_whatsapp_12
  If2_14 -->|main | out 2| Send_message_on_whatsapp1_16
  Switch1_15 -->|main| Expense_agent_6
  Switch1_15 -->|main | out 2| Payment_agent_4
  Switch1_15 -->|main | out 3| Invoice_agent_3
  Switch1_15 -->|main | out 4| Report_agent_5
  Send_message_on_whatsapp1_16 -->|main| Switch1_15
  Get_state_17 -->|main| If1_8
  Create_TRN_for_zoho_20 -.->|ai_tool| Zoho_TRN_24
  Create_TRN_for_zoho_20 -.->|ai_tool | item 2| AI_Agent_9
  Skip_TRN_for_Zoho_21 -.->|ai_tool| Zoho_TRN_24
  is_it_zoho_22 -->|main| Zoho_TRN_24
  is_it_zoho_22 -->|main | out 2| wafeq_TRN1_29
  OpenAI_Chat_Model3_23 -.->|ai_languageModel| wafeq_TRN1_29
  OpenAI_Chat_Model3_23 -.->|ai_languageModel | item 2| Zoho_TRN_24
  is_user_skipping_TRN_25 -->|main| Check_VAT_TRN_health_50
  is_user_skipping_TRN_25 -->|main | item 2| No_Operation__do_nothing_31
  is_user_skipping_TRN_25 -->|main | out 2| Call__check_VAT_TRN_health__40
  is_user_skipping_TRN_25 -->|main | out 2 | item 2| is_it_zoho_22
  Org_ID_26 -->|main| TRN_32
  Org_ID_26 -->|main | out 2| Onboarding_agent_2
  Simple_Memory_27 -.->|ai_memory| wafeq_TRN1_29
  Simple_Memory_27 -.->|ai_memory | item 2| Zoho_TRN_24
  Zoho_TRN_24 -->|main| Send_message_on_whatsapp2_28
  wafeq_TRN1_29 -->|main| Send_message_on_whatsapp2_28
  Skip_TRN_for_Wafeq_30 -.->|ai_tool| wafeq_TRN1_29
  No_Operation__do_nothing_31 -->|main| Get_state_17
  TRN_32 -->|main| No_Operation__do_nothing_31
  TRN_32 -->|main | item 2| Call__check_VAT_TRN_health_2_49
  TRN_32 -->|main | out 2| Get_row_s__38
  Expense_agent_6 -->|main | out 2| 0050000_34
  Payment_agent_4 -->|main | out 2| 0030000_35
  Invoice_agent_3 -->|main | out 2| 0020000_36
  Report_agent_5 -->|main | out 2| 0040000_37
  Get_row_s__38 -->|main| Limit_39
  Limit_39 -->|main| is_user_skipping_TRN_25
  When_clicking__Execute_workflow__42 -->|main| Edit_Fields_48
  HTTP_Request_test_name_43 -->|main | out 2| creating_a_task1_44
  zoho_46 -->|main| Limit1_45
  wafeq_47 -->|main| Limit1_45
  Edit_Fields_48 -->|main| zoho_46
  Edit_Fields_48 -->|main | item 2| wafeq_47
  Send_message_on_whatsapp_12 -->|main| Edit_Fields4_53
  Transcribe_a_recording_54 -->|main| Received_Values_56
  Is_it_Audio_55 -->|main| Transcribe_a_recording_54
  Is_it_Audio_55 -->|main | out 2| Switch3_68
  Received_Values_56 -->|main| Is_it_Audio_55
  Analyze_document_57 -->|main| Parsing_output_60
  Parsing_output_60 -->|main| expense_or_bankStatement_62
  expense_or_bankStatement_62 -->|main| Get_data_to_check_if_its_first_time_77
  expense_or_bankStatement_62 -->|main | out 2| Upsert_bankStatement_data_63
  HTTP_Request_65 -->|main| Extract_from_File_64
  HTTP_Request_65 -->|main | out 2| Extract_from_File_64
  Extract_from_File_64 -->|main| Limit2_70
  Message_a_model_66 -->|main| Upsert_bankStatement_data1_67
  Switch3_68 -->|main| Analyze_document_57
  Switch3_68 -->|main | out 2| HTTP_Request_65
  Switch3_68 -->|main | out 3| Message_injection_71
  Upsert_expense_data_61 -->|main| Message_update_72
  Upsert_bankStatement_data_63 -->|main| No_Operation__do_nothing1_69
  Limit2_70 -->|main| Message_a_model_66
  Upsert_bankStatement_data1_67 -->|main| No_Operation__do_nothing1_69
  Upsert_bankStatement_data1_67 -->|main | item 2| Get_accounts1_74
  No_Operation__do_nothing1_69 -->|main| Message_update_72
  Message_update_72 -->|main| Message_injection_71
  Message_injection_71 -->|main| Org_ID_26
  Get_accounts1_74 -->|main| HTTP_Request1_73
  If_75 -->|main| Upsert_expense_data2_76
  If_75 -->|main | out 2| Upsert_expense_data_61
  Upsert_expense_data2_76 -->|main| Message_update_72
  Get_data_to_check_if_its_first_time_77 -->|main| If_75
```
