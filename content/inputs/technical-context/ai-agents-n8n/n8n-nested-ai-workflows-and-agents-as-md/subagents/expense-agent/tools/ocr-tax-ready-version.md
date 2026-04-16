---
title: "OCR Tax ready version"
publish: true
---

# OCR Tax ready version

Source: `workflows/remote/dev_4/subagents/Expense agent V3/tools/OCR Tax ready version.json`

Purpose:
Performs OCR-based document extraction with explicit VAT/TRN interpretation and vendor handling.

Technical flow:
The workflow fetches and extracts file contents, uses AI models to analyze the document, checks whether TRN values exist and are valid, determines whether the vendor is charging tax, looks up tax-rate state in data tables, and builds a tax-ready summary of the OCR result. It also tries to match or create the vendor and stores the vendor identifier for later accounting actions.

Outputs:
The result is a structured OCR payload that includes invoice/expense fields, vendor linkage, and tax-readiness flags suitable for downstream expense recording.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  Extract_from_File_2["Extract from File"]
  HTTP_Request_3["HTTP Request"]
  Message_a_model_4["Message a model"]
  Send_message_on_whatsapp_5["Send message on whatsapp"]
  0050101_6["0050101"]
  0050102_7["0050102"]
  Sticky_Note_8["Sticky Note"]
  Sticky_Note1_9["Sticky Note1"]
  Sticky_Note5_10["Sticky Note5"]
  Sticky_Note6_11["Sticky Note6"]
  Sticky_Note4_12["Sticky Note4"]
  tax_rate_is_5_13["tax rate is 5"]
  If_15_digits_and_starts_with_1_14["If 15 digits and starts with 1"]
  Vendor_is_taxing_me_15["Vendor_is_taxing_me"]
  Vendor_is_taxing_me1_16["Vendor_is_taxing_me1"]
  TAX_rate_Info2_17["TAX rate Info2"]
  Sticky_Note11_18["Sticky Note11"]
  Skiped_tax_rate1_19["Skiped tax rate1"]
  TRN_is_skipped_20["TRN is skipped"]
  Getting_right_tax_rate_to_state_21["Getting right tax rate to state"]
  Sticky_Note12_22["Sticky Note12"]
  Sticky_Note13_23["Sticky Note13"]
  update_with_OCR_data_24["update with OCR data"]
  Check_TRN_on_invoice_25["Check TRN on invoice"]
  invoice_issuer_TRN_26["invoice_issuer_TRN"]
  Analyze_document_27["Analyze document"]
  List_files_28["List files"]
  Edit_Fields5_29["Edit Fields5"]
  recipient_TRN_30["recipient_TRN"]
  Sticky_Note14_31["Sticky Note14"]
  Sticky_Note15_32["Sticky Note15"]
  Sticky_Note16_33["Sticky Note16"]
  Not_UAE_Ready_34["Not UAE Ready"]
  5__UAE_Ready_35["5% UAE Ready"]
  Summary_of_Tax_options_36["Summary of Tax options"]
  OCR_matching_old_ocr_37["OCR matching old ocr"]
  OCR_is_done_tag_38["OCR is done tag"]
  output_not_clear_39["output not clear"]
  Sticky_Note9_40["Sticky Note9"]
  is_it_paid_41["is it paid"]
  Payment_accounts_42["Payment accounts"]
  Sticky_Note2_43["Sticky Note2"]
  Match_and_here_is_id_44["Match and here is id"]
  match_vendor_and_get_id_45["match vendor and get id"]
  create_46["create"]
  Edit_Fields_47["Edit Fields"]
  adding_vendor_new_id_48["adding vendor new id"]
  Saving_exciting_vendor_id_49["Saving exciting vendor id"]
  back_to_agent_output_50["back to agent output"]
  Sticky_Note3_51["Sticky Note3"]
  Sticky_Note7_52["Sticky Note7"]
  Sticky_Note8_53["Sticky Note8"]
  Sticky_Note10_54["Sticky Note10"]
  When_Executed_by_Another_Workflow_1 -->|main| Send_message_on_whatsapp_5
  When_Executed_by_Another_Workflow_1 -->|main | item 2| Analyze_document_27
  Extract_from_File_2 -->|main| Message_a_model_4
  HTTP_Request_3 -->|main| Extract_from_File_2
  HTTP_Request_3 -->|main | item 2| List_files_28
  HTTP_Request_3 -->|main | out 2| 0050101_6
  tax_rate_is_5_13 -->|main| Vendor_is_taxing_me_15
  tax_rate_is_5_13 -->|main | out 2| Vendor_is_taxing_me1_16
  If_15_digits_and_starts_with_1_14 -->|main| 5__UAE_Ready_35
  If_15_digits_and_starts_with_1_14 -->|main | out 2| Not_UAE_Ready_34
  Vendor_is_taxing_me_15 -->|main| invoice_issuer_TRN_26
  Vendor_is_taxing_me1_16 -->|main| Not_UAE_Ready_34
  TAX_rate_Info2_17 -->|main| TRN_is_skipped_20
  TRN_is_skipped_20 -->|main| Skiped_tax_rate1_19
  TRN_is_skipped_20 -->|main | out 2| Getting_right_tax_rate_to_state_21
  Getting_right_tax_rate_to_state_21 -->|main| tax_rate_is_5_13
  update_with_OCR_data_24 -->|main| TAX_rate_Info2_17
  Check_TRN_on_invoice_25 -->|main| If_15_digits_and_starts_with_1_14
  Skiped_tax_rate1_19 -->|main| Not_UAE_Ready_34
  invoice_issuer_TRN_26 -->|main| recipient_TRN_30
  invoice_issuer_TRN_26 -->|main | out 2| Not_UAE_Ready_34
  Analyze_document_27 -->|main| OCR_matching_old_ocr_37
  recipient_TRN_30 -->|main| Check_TRN_on_invoice_25
  recipient_TRN_30 -->|main | out 2| Not_UAE_Ready_34
  Not_UAE_Ready_34 -->|main| Summary_of_Tax_options_36
  5__UAE_Ready_35 -->|main| Summary_of_Tax_options_36
  Summary_of_Tax_options_36 -->|main| match_vendor_and_get_id_45
  OCR_matching_old_ocr_37 -->|main| update_with_OCR_data_24
  OCR_is_done_tag_38 -->|main| back_to_agent_output_50
  output_not_clear_39 -->|main| OCR_is_done_tag_38
  Match_and_here_is_id_44 -->|main| Saving_exciting_vendor_id_49
  Match_and_here_is_id_44 -->|main | out 2| create_46
  match_vendor_and_get_id_45 -->|main| Match_and_here_is_id_44
  create_46 -->|main| adding_vendor_new_id_48
  create_46 -->|main | out 2| Edit_Fields_47
  Saving_exciting_vendor_id_49 -->|main| output_not_clear_39
  adding_vendor_new_id_48 -->|main| output_not_clear_39
```
