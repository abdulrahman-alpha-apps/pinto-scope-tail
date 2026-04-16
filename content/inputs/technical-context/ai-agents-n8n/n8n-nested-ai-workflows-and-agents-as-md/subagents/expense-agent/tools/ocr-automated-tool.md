---
title: "OCR automated tool"
publish: true
---

# OCR automated tool

Source: `workflows/remote/dev_4/subagents/Expense agent V3/tools/OCR automated tool.json`

Purpose:
Main automated extraction workflow for expense files.

Technical flow:
This workflow validates the extracted document fields, classifies tax treatment, checks TRN patterns, pulls chart-of-account data, and uses an AI agent plus structured output parser to turn raw OCR into accounting-ready fields. It also handles vendor matching/creation, payment-state detection, and cash-payment updates.

Why it matters:
This is the heavy-lifting automation behind the expense agent. It converts a raw uploaded bill or receipt into a cleaned expense object with enough metadata for posting into the accounting system.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  Send_message_on_whatsapp_2["Send message on whatsapp"]
  Sticky_Note_3["Sticky Note"]
  Sticky_Note1_4["Sticky Note1"]
  Sticky_Note4_5["Sticky Note4"]
  tax_rate_is_5_6["tax rate is 5"]
  If_15_digits_and_starts_with_1_7["If 15 digits and starts with 1"]
  TAX_rate_Info2_8["TAX rate Info2"]
  Skiped_tax_rate1_9["Skiped tax rate1"]
  TRN_is_skipped_10["TRN is skipped"]
  Getting_right_tax_rate_to_state_11["Getting right tax rate to state"]
  Sticky_Note12_12["Sticky Note12"]
  Sticky_Note13_13["Sticky Note13"]
  Check_TRN_on_invoice_14["Check TRN on invoice"]
  Sticky_Note15_15["Sticky Note15"]
  Sticky_Note16_16["Sticky Note16"]
  Not_UAE_Ready_17["Not UAE Ready"]
  5__UAE_Ready_18["5% UAE Ready"]
  Summary_of_Tax_options_19["Summary of Tax options"]
  OCR_is_done_tag_20["OCR is done tag"]
  Sticky_Note2_21["Sticky Note2"]
  Match_and_here_is_id_22["Match and here is id"]
  match_vendor_and_get_id_23["match vendor and get id"]
  create_24["create"]
  Edit_Fields_25["Edit Fields"]
  adding_vendor_new_id_26["adding vendor new id"]
  Saving_exciting_vendor_id_27["Saving exciting vendor id"]
  back_to_agent_output_28["back to agent output"]
  Sticky_Note3_29["Sticky Note3"]
  Sticky_Note7_30["Sticky Note7"]
  Sticky_Note8_31["Sticky Note8"]
  Sticky_Note10_32["Sticky Note10"]
  Required_fields_for_check_33["Required fields for check"]
  Fields_check_34["Fields check"]
  OpenAI_Chat_Model_35["OpenAI Chat Model"]
  Filter_COA_36["Filter COA"]
  AI_Agent_37["AI Agent"]
  Structured_Output_Parser_38["Structured Output Parser"]
  xAI_Grok_Chat_Model_39["xAI Grok Chat Model"]
  Sticky_Note14_40["Sticky Note14"]
  adding_vendor_new_id1_41["adding vendor new id1"]
  YesVendorTaxing_42["YesVendorTaxing"]
  NoVendorTaxing_43["NoVendorTaxing"]
  Get_CoA_44["Get CoA"]
  Code_in_JavaScript_45["Code in JavaScript"]
  is_it_Paid_46["is it Paid"]
  Cash_47["Cash"]
  Updating_cash_payment_48["Updating cash payment"]
  Sticky_Note17_49["Sticky Note17"]
  If_50["If"]
  Bill_is_invalid_51["Bill is invalid"]
  Sticky_Note5_52["Sticky Note5"]
  Sticky_Note6_53["Sticky Note6"]
  Sticky_Note9_54["Sticky Note9"]
  Sticky_Note11_55["Sticky Note11"]
  Sticky_Note18_56["Sticky Note18"]
  Get_row_s__57["Get row(s)"]
  YesVendorTaxing1_58["YesVendorTaxing1"]
  YesVendorTaxing2_59["YesVendorTaxing2"]
  When_Executed_by_Another_Workflow_1 -->|main| Send_message_on_whatsapp_2
  When_Executed_by_Another_Workflow_1 -->|main | item 2| Get_row_s__57
  tax_rate_is_5_6 -->|main| YesVendorTaxing_42
  tax_rate_is_5_6 -->|main | out 2| NoVendorTaxing_43
  If_15_digits_and_starts_with_1_7 -->|main| 5__UAE_Ready_18
  If_15_digits_and_starts_with_1_7 -->|main | out 2| Not_UAE_Ready_17
  TAX_rate_Info2_8 -->|main| TRN_is_skipped_10
  Skiped_tax_rate1_9 -->|main| Not_UAE_Ready_17
  TRN_is_skipped_10 -->|main| Skiped_tax_rate1_9
  TRN_is_skipped_10 -->|main | out 2| Getting_right_tax_rate_to_state_11
  Getting_right_tax_rate_to_state_11 -->|main| tax_rate_is_5_6
  Check_TRN_on_invoice_14 -->|main| If_15_digits_and_starts_with_1_7
  Not_UAE_Ready_17 -->|main| Summary_of_Tax_options_19
  5__UAE_Ready_18 -->|main| Summary_of_Tax_options_19
  Summary_of_Tax_options_19 -->|main| match_vendor_and_get_id_23
  OCR_is_done_tag_20 -->|main| back_to_agent_output_28
  Match_and_here_is_id_22 -->|main| Saving_exciting_vendor_id_27
  Match_and_here_is_id_22 -->|main | out 2| create_24
  match_vendor_and_get_id_23 -->|main| Match_and_here_is_id_22
  create_24 -->|main| adding_vendor_new_id_26
  create_24 -->|main | out 2| Edit_Fields_25
  adding_vendor_new_id_26 -->|main| Get_CoA_44
  Saving_exciting_vendor_id_27 -->|main| Get_CoA_44
  Required_fields_for_check_33 -->|main| Fields_check_34
  Fields_check_34 -->|main| Check_TRN_on_invoice_14
  Fields_check_34 -->|main | out 2| Not_UAE_Ready_17
  OpenAI_Chat_Model_35 -.->|ai_languageModel| AI_Agent_37
  Filter_COA_36 -->|main| AI_Agent_37
  AI_Agent_37 -->|main| adding_vendor_new_id1_41
  Structured_Output_Parser_38 -.->|ai_outputParser| AI_Agent_37
  adding_vendor_new_id1_41 -->|main| is_it_Paid_46
  YesVendorTaxing_42 -->|main| Required_fields_for_check_33
  NoVendorTaxing_43 -->|main| Not_UAE_Ready_17
  Get_CoA_44 -->|main| Filter_COA_36
  is_it_Paid_46 -->|main| Cash_47
  is_it_Paid_46 -->|main | out 2| OCR_is_done_tag_20
  Cash_47 -->|main | out 2| OCR_is_done_tag_20
  Code_in_JavaScript_45 -->|main| Updating_cash_payment_48
  Updating_cash_payment_48 -->|main| OCR_is_done_tag_20
  If_50 -->|main| TAX_rate_Info2_8
  If_50 -->|main | out 2| YesVendorTaxing1_58
  Get_row_s__57 -->|main| If_50
  YesVendorTaxing1_58 -->|main| Bill_is_invalid_51
  Edit_Fields_25 -->|main| YesVendorTaxing2_59
```
