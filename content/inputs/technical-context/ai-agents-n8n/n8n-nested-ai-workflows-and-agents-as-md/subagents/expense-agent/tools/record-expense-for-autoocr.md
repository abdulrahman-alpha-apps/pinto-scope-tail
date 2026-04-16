---
title: "Record expense for autoOCR"
publish: true
---

# Record expense for autoOCR

Source: `workflows/remote/dev_4/subagents/Expense agent V3/tools/Record expense for autoOCR.json`

Purpose:
Creates the final accounting expense after OCR extraction has finished.

Technical flow:
The workflow branches on paid vs unpaid expense state, vendor tax behavior, and cash-payment handling. It builds or removes payment and tax fields as needed, calls different HTTP endpoints for the correct posting pattern, updates thread status records, and prepares user-facing confirmation messages for the main agent.

Why it exists:
OCR extraction and actual accounting posting are separated. This tool is the posting stage that turns extracted expense data into a real accounting transaction.

## Diagram

```mermaid
flowchart TD
  When_Executed_by_Another_Workflow_1["When Executed by Another Workflow"]
  Record_Unpaid_expense_2["Record Unpaid expense"]
  Record_paid_expense_3["Record paid expense"]
  Send_message_on_whatsapp_4["Send message on whatsapp"]
  Remove_Payment_Data_5["Remove Payment Data"]
  Message_to_Main_Agent_6["Message to Main Agent"]
  Message_to_Main_Agent1_7["Message to Main Agent1"]
  0050601_8["0050601"]
  0050602_9["0050602"]
  Build_paidData_JSON_10["Build paidData JSON"]
  Record_Unpaid_expense1_11["Record Unpaid expense1"]
  Edit_Fields_12["Edit Fields"]
  is_it_paid_13["is it paid"]
  vendor_is_taxing_me_14["vendor is taxing me"]
  Record_Unpaid_expense2_15["Record Unpaid expense2"]
  Edit_Fields1_16["Edit Fields1"]
  vendor_is_taxing_me1_17["vendor is taxing me1"]
  Edit_Fields2_18["Edit Fields2"]
  Sticky_Note_19["Sticky Note"]
  Sticky_Note1_20["Sticky Note1"]
  Sticky_Note2_21["Sticky Note2"]
  Sticky_Note3_22["Sticky Note3"]
  Sticky_Note4_23["Sticky Note4"]
  Sticky_Note5_24["Sticky Note5"]
  Removing_tax_rate_ID_25["Removing tax rate ID"]
  Remove_tax_rate_ID_26["Remove tax rate ID"]
  Edit_Fields3_27["Edit Fields3"]
  Switch_28["Switch"]
  Edit_Fields4_29["Edit Fields4"]
  Sticky_Note7_30["Sticky Note7"]
  Sticky_Note8_31["Sticky Note8"]
  Sticky_Note9_32["Sticky Note9"]
  IsItCashPaid_33["IsItCashPaid"]
  Message_to_Main_Agent2_34["Message to Main Agent2"]
  Message_to_Main_Agent3_35["Message to Main Agent3"]
  Update_thread_status_36["Update thread status"]
  Update_thread_status1_37["Update thread status1"]
  Update_thread_status2_38["Update thread status2"]
  Update_thread_status3_39["Update thread status3"]
  hadObjection_40["hadObjection"]
  When_Executed_by_Another_Workflow_1 -->|main| Send_message_on_whatsapp_4
  When_Executed_by_Another_Workflow_1 -->|main | item 2| Build_paidData_JSON_10
  Remove_Payment_Data_5 -->|main| vendor_is_taxing_me_14
  Remove_Payment_Data_5 -->|main | out 2| 0050602_9
  Record_paid_expense_3 -->|main| Update_thread_status1_37
  Record_paid_expense_3 -->|main | out 2| Switch_28
  Record_Unpaid_expense_2 -->|main| Update_thread_status_36
  Record_Unpaid_expense_2 -->|main | out 2| Edit_Fields3_27
  Build_paidData_JSON_10 -->|main| is_it_paid_13
  Edit_Fields_12 -->|main| Removing_tax_rate_ID_25
  is_it_paid_13 -->|main| IsItCashPaid_33
  is_it_paid_13 -->|main | out 2| Remove_Payment_Data_5
  vendor_is_taxing_me_14 -->|main| Record_Unpaid_expense1_11
  vendor_is_taxing_me_14 -->|main | out 2| Edit_Fields_12
  Edit_Fields1_16 -->|main| Remove_tax_rate_ID_26
  vendor_is_taxing_me1_17 -->|main| Edit_Fields2_18
  vendor_is_taxing_me1_17 -->|main | out 2| Edit_Fields1_16
  Record_Unpaid_expense1_11 -->|main| Update_thread_status_36
  Record_Unpaid_expense1_11 -->|main | out 2| Switch_28
  Edit_Fields2_18 -->|main| Record_paid_expense_3
  Removing_tax_rate_ID_25 -->|main| Record_Unpaid_expense_2
  Remove_tax_rate_ID_26 -->|main| Record_Unpaid_expense2_15
  Record_Unpaid_expense2_15 -->|main| Update_thread_status2_38
  Record_Unpaid_expense2_15 -->|main | out 2| Edit_Fields3_27
  Switch_28 -->|main| Edit_Fields4_29
  IsItCashPaid_33 -->|main| vendor_is_taxing_me1_17
  IsItCashPaid_33 -->|main | out 2| Remove_Payment_Data_5
  Update_thread_status_36 -->|main| Message_to_Main_Agent1_7
  Update_thread_status1_37 -->|main| Message_to_Main_Agent_6
  Update_thread_status2_38 -->|main| Message_to_Main_Agent2_34
  Edit_Fields4_29 -->|main| hadObjection_40
  hadObjection_40 -->|main| Update_thread_status3_39
```
