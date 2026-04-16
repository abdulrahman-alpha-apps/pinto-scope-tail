---
title: "2. All Create Task Cases"
publish: true
---

# All Create Task Cases — Human-in-the-Loop

## API Reference

**Endpoint:** `POST /ai/organizations/tasks`
**Auth:** `aiApiKey`
**Returns:** `AccountantTask`

### CreateTaskDto Schema

```json
{
  "metadata": {                   // required — agent context
    "threadId": "string",         // required
    "userId": "string"            // required
  },
  "title": "string",              // required — short summary shown on task card
  "description": "string",        // required — full context for accountant
  "sourceType": "ai_agent",       // required — always "ai_agent" for these cases
  "category": "issue | normal_task | error | user_objection",  // required
  "entity": "invoice | bill_ocr | bill_manual_entry | payment | report | general_task",  // optional
  "data": { },                    // optional — pre-filled extracted data
  "attachment": "string",         // optional — URL to source document
  "errorCode": "string"           // optional — machine-readable error identifier
}
```

> `user_objection` is a new category value — requires backend schema update before agents can use it.

---

## How to Read This Document

Each case below defines:
- **Trigger condition** — what causes the task to be created
- **Agent / Tool** — which n8n workflow fires the call
- **Request body** — the exact JSON sent to the API for that case
- **Thread status after** — what state the thread is set to after task creation
- **User WhatsApp message** — what the user is told

---

## Agent 1: Router Agent

The Router is the system entry point. It receives every incoming message and routes it to a specialist agent. Task creation at the Router level means the system could not even begin to process the request.

---

### Case R-1: Unrecognized or unroutable intent after retries

**Trigger:** The Router AI Agent node loops or produces an intent classification the Switch node cannot match, and a retry threshold is reached.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Message could not be routed",
  "description": "The user sent a message that the Router agent was unable to classify and route after multiple attempts. The message content is preserved in the data field for accountant review.",
  "sourceType": "ai_agent",
  "category": "normal_task",
  "entity": "general_task",
  "data": {
    "rawMessage": "{{originalUserMessage}}",
    "detectedIntent": "{{lastIntentAttempt}}",
    "attemptCount": "{{retryCount}}"
  },
  "errorCode": "ROUTER_UNRESOLVABLE_INTENT"
}
```

**Thread status after:** `escalated`
**User message:** *"We received your message but need a moment to process it. Our team will follow up shortly."*

---

### Case R-2: Webhook or system-level processing crash

**Trigger:** An unhandled exception in the Router workflow — Webhook node, Code node, or downstream HTTP call — causes the pipeline to fail before any agent is reached.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "System error — Router pipeline failed",
  "description": "A processing error occurred in the Router workflow before the message could be dispatched to a specialist agent. This may be a transient infrastructure issue or a malformed incoming payload.",
  "sourceType": "ai_agent",
  "category": "error",
  "entity": "general_task",
  "data": {
    "rawMessage": "{{originalUserMessage}}",
    "errorDetails": "{{errorMessage}}"
  },
  "errorCode": "ROUTER_PIPELINE_CRASH"
}
```

**Thread status after:** `escalated`
**User message:** *"We encountered an issue processing your request. Our team has been notified and will follow up."*

---

### Case R-3: TRN / VAT setup failure during onboarding routing

**Trigger:** The Router's VAT/TRN branch (Check VAT TRN health, Create TRN for Zoho, Skip TRN workflows) fails after retries — the VAT configuration cannot be confirmed or created.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "VAT/TRN setup could not be completed",
  "description": "The VAT or TRN configuration step failed during routing. The user's tax registration number could not be validated or created in the connected accounting system.",
  "sourceType": "ai_agent",
  "category": "issue",
  "entity": "general_task",
  "data": {
    "accountingProvider": "{{provider}}",
    "trnValue": "{{trnAttempted}}",
    "healthCheckResult": "{{healthCheckOutput}}"
  },
  "errorCode": "ROUTER_TRN_SETUP_FAILED"
}
```

**Thread status after:** `escalated`
**User message:** *"We couldn't complete your VAT setup automatically. Our team will review and get this sorted for you."*

---

## Agent 2: Expense Agent

The Expense Agent handles receipt/bill processing via OCR and manual input. It already has a `Fallback dummy` tool for escalation. The cases below define the complete trigger matrix and standardize the request body for each.

---

### Case E-1: OCR extraction failed — mandatory fields missing after 3 attempts

**Trigger:** The OCR automated tool runs but the extracted output is missing required fields (vendor, amount, date). The agent asks the user to resend and reaches the attempt limit.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Expense OCR failed — mandatory fields missing",
  "description": "OCR extraction was attempted 3 times but could not extract all required fields from the uploaded document. Manual review is needed to complete the expense entry.",
  "sourceType": "ai_agent",
  "category": "issue",
  "entity": "bill_ocr",
  "attachment": "{{documentUrl}}",
  "data": {
    "extractedVendor": "{{vendorOrNull}}",
    "extractedAmount": "{{amountOrNull}}",
    "extractedDate": "{{dateOrNull}}",
    "extractedTaxAmount": "{{taxAmountOrNull}}",
    "extractedTRN": "{{trnOrNull}}",
    "attemptCount": 3,
    "missingFields": ["{{field1}}", "{{field2}}"]
  },
  "errorCode": "EXPENSE_OCR_FIELDS_MISSING"
}
```

**Thread status after:** `escalated`
**User message:** *"We couldn't read your document clearly. Our team will review and complete the entry for you."*

---

### Case E-2: Vendor mismatch — not resolved after user prompts

**Trigger:** OCR or manual input returns a vendor name. The Match Vendor tool finds candidates but the confidence is too low or the user's confirmation does not resolve the ambiguity after retries.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Expense vendor could not be confirmed",
  "description": "The vendor name from the uploaded document could not be matched to an existing contact with sufficient confidence, and the user's responses did not resolve the ambiguity.",
  "sourceType": "ai_agent",
  "category": "issue",
  "entity": "bill_ocr",
  "attachment": "{{documentUrl}}",
  "data": {
    "extractedVendorName": "{{rawVendorName}}",
    "candidateMatches": [
      { "id": "{{contactId}}", "name": "{{contactName}}", "score": "{{matchScore}}" }
    ],
    "extractedAmount": "{{amount}}",
    "extractedDate": "{{date}}",
    "currency": "{{currency}}",
    "taxAmount": "{{taxAmount}}"
  },
  "errorCode": "EXPENSE_VENDOR_MATCH_FAILED"
}
```

**Thread status after:** `escalated`
**User message:** *"We couldn't confirm your vendor automatically. Our team will review and complete the entry."*

---

### Case E-3: TRN validation failed — bill invalid flag

**Trigger:** OCR automated tool sets the `Bill is invalid` path — the TRN on the document does not pass FTA format check (not 15 digits starting with 1) and the expense cannot be posted with the incorrect TRN.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Expense bill invalid — TRN validation failed",
  "description": "The supplier TRN extracted from the document failed the FTA format check. The bill cannot be posted with an invalid TRN. Accountant should verify the TRN and decide on the correct tax treatment.",
  "sourceType": "ai_agent",
  "category": "issue",
  "entity": "bill_ocr",
  "attachment": "{{documentUrl}}",
  "data": {
    "extractedTRN": "{{trnValue}}",
    "vendorName": "{{vendorName}}",
    "amount": "{{amount}}",
    "taxAmount": "{{taxAmount}}",
    "trnValidationResult": "failed"
  },
  "errorCode": "EXPENSE_TRN_INVALID"
}
```

**Thread status after:** `escalated`
**User message:** *"There was an issue with the tax number on your document. Our team will review and complete this for you."*

---

### Case E-4: Chart of Accounts — category not resolved

**Trigger:** Manual input or OCR flow runs but the Selecting Account tool (chart of accounts) cannot find a suitable category and the user does not confirm a suggestion within the retry window.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Expense account category not confirmed",
  "description": "The expense agent could not determine the correct chart of accounts category for this transaction. User confirmation attempts were exhausted.",
  "sourceType": "ai_agent",
  "category": "issue",
  "entity": "bill_manual_entry",
  "data": {
    "vendorName": "{{vendorName}}",
    "amount": "{{amount}}",
    "currency": "{{currency}}",
    "suggestedCategory": "{{suggestedAccountName}}",
    "suggestedCategoryId": "{{suggestedAccountId}}",
    "userDescription": "{{userProvidedDescription}}"
  },
  "errorCode": "EXPENSE_ACCOUNT_UNRESOLVED"
}
```

**Thread status after:** `escalated`
**User message:** *"We need a bit more help categorizing this expense. Our team will take a look and sort it out."*

---

### Case E-5: Accounting system API failure during expense posting

**Trigger:** The Record Expense tool calls the backend (which calls Wafeq or Zoho) and receives a non-recoverable error response after retries.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Expense posting failed — accounting system error",
  "description": "The expense data was extracted and confirmed but the posting to the accounting system failed with an API error. The extracted data is preserved in full for manual posting.",
  "sourceType": "ai_agent",
  "category": "error",
  "entity": "bill_ocr",
  "attachment": "{{documentUrl}}",
  "data": {
    "vendorId": "{{vendorId}}",
    "vendorName": "{{vendorName}}",
    "amount": "{{amount}}",
    "taxAmount": "{{taxAmount}}",
    "taxRate": "{{taxRate}}",
    "currency": "{{currency}}",
    "date": "{{billDate}}",
    "accountId": "{{chartOfAccountId}}",
    "paymentAccountId": "{{paymentAccountId}}",
    "billNumber": "{{billNumber}}",
    "apiErrorCode": "{{providerErrorCode}}",
    "apiErrorMessage": "{{providerErrorMessage}}"
  },
  "errorCode": "EXPENSE_POST_API_FAILURE"
}
```

**Thread status after:** `escalated`
**User message:** *"We ran into a technical issue recording your expense. Our team has been notified and will complete it for you."*

---

### Case E-6: User objection — disputes VAT treatment applied by agent

**Trigger:** The agent applied a VAT treatment (e.g., marked the expense as reclaimable input VAT at 5%) and the user explicitly responds that this is wrong — for example, the user says they want it as non-reclaimable, zero-rated, or a different tax category.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "User disputes VAT treatment on expense",
  "description": "The user explicitly objected to the VAT treatment the agent applied. The agent set the tax as reclaimable but the user wants a different treatment. Accountant must review the correct VAT classification and apply it before posting.",
  "sourceType": "ai_agent",
  "category": "user_objection",
  "entity": "bill_ocr",
  "attachment": "{{documentUrl}}",
  "data": {
    "vendorName": "{{vendorName}}",
    "amount": "{{amount}}",
    "agentAppliedTaxRate": "{{agentTaxRate}}",
    "agentAppliedTaxType": "{{agentTaxType}}",
    "userRequestedTaxType": "{{userDescribedPreference}}",
    "userMessage": "{{verbatimUserMessage}}"
  },
  "errorCode": "EXPENSE_USER_VAT_OBJECTION"
}
```

**Thread status after:** `escalated`
**User message:** *"Understood — we've flagged this for our team to apply the correct VAT treatment and complete your entry."*

---

### Case E-7: User objection — disputes expense category or vendor

**Trigger:** The agent posted or confirmed an expense category or vendor and the user replies to say it is wrong and wants it changed.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "User disputes expense category or vendor",
  "description": "The user objected to the account category or vendor the agent applied. The accountant must review the correct classification and update the record.",
  "sourceType": "ai_agent",
  "category": "user_objection",
  "entity": "bill_manual_entry",
  "data": {
    "agentAppliedCategory": "{{categoryName}}",
    "agentAppliedVendor": "{{vendorName}}",
    "userRequestedChange": "{{userDescribedChange}}",
    "userMessage": "{{verbatimUserMessage}}",
    "existingRecordId": "{{wafeqOrZohoBillId}}"
  },
  "errorCode": "EXPENSE_USER_CATEGORY_OBJECTION"
}
```

**Thread status after:** `escalated`
**User message:** *"Got it — our team will update this with the correct details."*

---

## Agent 3: Invoice Agent

The Invoice Agent handles invoice creation, customer and item matching, and delivery. It currently has an `exit` tool that closes conversations cleanly but does not create an accountant task. A new HITL fallback tool is needed.

---

### Case I-1: Customer not found or created — API failure

**Trigger:** Match Customer tool finds no match, Create Customer tool is called, and the backend API call to create the contact in Wafeq/Zoho fails.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Invoice customer could not be created",
  "description": "The invoice agent matched no existing customer for the provided name and the attempt to create a new customer contact in the accounting system failed. Invoice creation is blocked.",
  "sourceType": "ai_agent",
  "category": "issue",
  "entity": "invoice",
  "data": {
    "customerNameProvided": "{{customerName}}",
    "invoiceItems": [
      { "description": "{{itemDesc}}", "quantity": "{{qty}}", "unitPrice": "{{price}}" }
    ],
    "currency": "{{currency}}",
    "invoiceDate": "{{date}}",
    "apiErrorMessage": "{{providerErrorMessage}}"
  },
  "errorCode": "INVOICE_CUSTOMER_CREATE_FAILED"
}
```

**Thread status after:** `escalated`
**User message:** *"We had trouble creating your customer record. Our team will complete your invoice."*

---

### Case I-2: Invoice item not resolved

**Trigger:** Match Items or Create Items tool fails to resolve one or more line items — either the item does not exist in the chart of accounts and creation fails, or there is ambiguity the agent cannot resolve.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Invoice item could not be resolved",
  "description": "One or more invoice line items could not be matched to an existing product or service in the accounting system, and the agent was unable to create them automatically.",
  "sourceType": "ai_agent",
  "category": "issue",
  "entity": "invoice",
  "data": {
    "customerId": "{{customerId}}",
    "customerName": "{{customerName}}",
    "unresolvedItems": [
      { "description": "{{itemDesc}}", "quantity": "{{qty}}", "unitPrice": "{{price}}" }
    ],
    "resolvedItems": [
      { "itemId": "{{itemId}}", "description": "{{itemDesc}}", "quantity": "{{qty}}", "unitPrice": "{{price}}" }
    ],
    "currency": "{{currency}}"
  },
  "errorCode": "INVOICE_ITEM_UNRESOLVED"
}
```

**Thread status after:** `escalated`
**User message:** *"We couldn't confirm all the items on your invoice. Our team will complete it for you."*

---

### Case I-3: Invoice posting failed — accounting system API error

**Trigger:** All invoice data is confirmed and the Create Invoice tool posts to the backend, but the Wafeq or Zoho API returns an error.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Invoice submission failed — accounting system error",
  "description": "The invoice data was fully confirmed but the submission to the accounting system returned an API error. All extracted data is preserved for manual posting.",
  "sourceType": "ai_agent",
  "category": "error",
  "entity": "invoice",
  "data": {
    "customerId": "{{customerId}}",
    "customerName": "{{customerName}}",
    "invoiceDate": "{{invoiceDate}}",
    "dueDate": "{{dueDate}}",
    "currency": "{{currency}}",
    "taxAmountType": "{{taxType}}",
    "lineItems": [
      {
        "itemId": "{{itemId}}",
        "description": "{{desc}}",
        "quantity": "{{qty}}",
        "unitPrice": "{{price}}",
        "taxRate": "{{taxRate}}"
      }
    ],
    "totalAmount": "{{total}}",
    "apiErrorCode": "{{providerErrorCode}}",
    "apiErrorMessage": "{{providerErrorMessage}}"
  },
  "errorCode": "INVOICE_POST_API_FAILURE"
}
```

**Thread status after:** `escalated`
**User message:** *"We hit a technical issue submitting your invoice. Our team has been notified and will complete it."*

---

### Case I-4: Invoice timeout — conversation idle before completion

**Trigger:** The Timeout Notifier workflow fires, the state check shows the invoice thread is still incomplete (not cleared), and the user does not respond to the timeout reminder.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Invoice flow timed out — incomplete conversation",
  "description": "The invoice creation conversation became idle before all required information was collected. The partial data is preserved below.",
  "sourceType": "ai_agent",
  "category": "normal_task",
  "entity": "invoice",
  "data": {
    "partialCustomerName": "{{customerNameOrNull}}",
    "partialItems": "{{itemsSoFarOrNull}}",
    "partialCurrency": "{{currencyOrNull}}",
    "lastUserMessage": "{{lastMessage}}",
    "idleMinutes": "{{minutesIdle}}"
  },
  "errorCode": "INVOICE_CONVERSATION_TIMEOUT"
}
```

**Thread status after:** `escalated`
**User message:** *"It looks like we didn't finish your invoice. Our team will follow up to complete it."*

---

### Case I-5: User objection — disputes invoice line items or VAT

**Trigger:** The agent confirmed the invoice and the user replies that something is wrong — line item descriptions, quantities, prices, or the VAT treatment applied are incorrect.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "User disputes invoice details",
  "description": "The user objected to one or more details on the invoice that the agent confirmed. The accountant must review the user's stated correction and update the invoice before it is finalised.",
  "sourceType": "ai_agent",
  "category": "user_objection",
  "entity": "invoice",
  "data": {
    "existingInvoiceId": "{{invoiceIdIfCreated}}",
    "customerName": "{{customerName}}",
    "agentConfirmedItems": [
      { "description": "{{desc}}", "quantity": "{{qty}}", "unitPrice": "{{price}}", "taxRate": "{{taxRate}}" }
    ],
    "userRequestedChange": "{{userDescribedChange}}",
    "userMessage": "{{verbatimUserMessage}}"
  },
  "errorCode": "INVOICE_USER_OBJECTION"
}
```

**Thread status after:** `escalated`
**User message:** *"Noted — our team will correct the details and finalise your invoice."*

---

## Agent 4: Payment Agent

The Payment Agent already has a `Payment Fallback` tool. The cases below define the complete trigger matrix.

---

### Case P-1: Contact not matched or created for payment

**Trigger:** Match Contact and Create Contact tools both fail — the payer/payee cannot be resolved against Wafeq or Zoho contacts.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Payment contact could not be resolved",
  "description": "The payment agent could not match or create the contact associated with this payment. The payment cannot be posted without a valid contact.",
  "sourceType": "ai_agent",
  "category": "issue",
  "entity": "payment",
  "data": {
    "contactNameProvided": "{{contactName}}",
    "paymentAmount": "{{amount}}",
    "paymentCurrency": "{{currency}}",
    "paymentDate": "{{date}}",
    "paymentMethod": "{{method}}",
    "candidateMatches": [
      { "id": "{{contactId}}", "name": "{{contactName}}", "score": "{{score}}" }
    ]
  },
  "errorCode": "PAYMENT_CONTACT_UNRESOLVED"
}
```

**Thread status after:** `escalated`
**User message:** *"We couldn't confirm the contact for this payment. Our team will review and complete the record."*

---

### Case P-2: Invoice or bill not found for payment allocation

**Trigger:** Get Invoice/Bill Details tool finds no open invoice or bill matching the user's payment reference, or there are multiple matches and the agent cannot determine which to apply it to.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Payment — matching invoice or bill not found",
  "description": "The payment agent could not identify the invoice or bill this payment should be applied to. The payment details are preserved for manual allocation.",
  "sourceType": "ai_agent",
  "category": "issue",
  "entity": "payment",
  "data": {
    "contactName": "{{contactName}}",
    "paymentAmount": "{{amount}}",
    "paymentCurrency": "{{currency}}",
    "paymentDate": "{{date}}",
    "paymentReference": "{{referenceOrNull}}",
    "candidateInvoices": [
      { "id": "{{invoiceId}}", "number": "{{invoiceNumber}}", "amount": "{{invoiceAmount}}", "dueDate": "{{dueDate}}" }
    ]
  },
  "errorCode": "PAYMENT_INVOICE_NOT_FOUND"
}
```

**Thread status after:** `escalated`
**User message:** *"We couldn't match your payment to an open invoice. Our team will allocate it correctly."*

---

### Case P-3: Bank statement extraction failed

**Trigger:** Extract Bank Statement File tool processes the uploaded CSV or PDF but cannot parse it into valid payment rows after retries.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Bank statement could not be parsed",
  "description": "The uploaded bank statement file could not be extracted into usable payment rows. The file is attached for manual review.",
  "sourceType": "ai_agent",
  "category": "issue",
  "entity": "payment",
  "attachment": "{{fileUrl}}",
  "data": {
    "fileType": "{{csv_or_pdf}}",
    "rowsExtracted": "{{rowCountOrZero}}",
    "parseErrorDetail": "{{errorDetail}}"
  },
  "errorCode": "PAYMENT_BANK_STATEMENT_PARSE_FAILED"
}
```

**Thread status after:** `escalated`
**User message:** *"We had trouble reading your bank statement file. Our team will process it manually."*

---

### Case P-4: Payment posting failed — accounting system API error

**Trigger:** Create Cash Payment tool posts to the backend and the Wafeq or Zoho API returns a non-recoverable error.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Payment posting failed — accounting system error",
  "description": "All payment details were confirmed but the accounting system returned an API error during posting. The full payment data is preserved for manual entry.",
  "sourceType": "ai_agent",
  "category": "error",
  "entity": "payment",
  "data": {
    "contactId": "{{contactId}}",
    "contactName": "{{contactName}}",
    "invoiceId": "{{invoiceId}}",
    "paymentAmount": "{{amount}}",
    "paymentCurrency": "{{currency}}",
    "paymentDate": "{{date}}",
    "paymentAccountId": "{{accountId}}",
    "paymentMethod": "{{method}}",
    "apiErrorCode": "{{providerErrorCode}}",
    "apiErrorMessage": "{{providerErrorMessage}}"
  },
  "errorCode": "PAYMENT_POST_API_FAILURE"
}
```

**Thread status after:** `escalated`
**User message:** *"We hit a technical issue recording your payment. Our team has been notified and will complete it."*

---

### Case P-5: User objection — disputes payment allocation or amount

**Trigger:** The agent recorded or proposed a payment allocation and the user replies that the amount, date, method, or invoice allocation is wrong.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "User disputes payment details",
  "description": "The user objected to how the payment was recorded or allocated by the agent. The accountant must review the correct allocation and update the record.",
  "sourceType": "ai_agent",
  "category": "user_objection",
  "entity": "payment",
  "data": {
    "existingPaymentId": "{{paymentIdIfPosted}}",
    "agentRecordedAmount": "{{amount}}",
    "agentRecordedDate": "{{date}}",
    "agentAllocatedInvoiceId": "{{invoiceId}}",
    "userRequestedChange": "{{userDescribedChange}}",
    "userMessage": "{{verbatimUserMessage}}"
  },
  "errorCode": "PAYMENT_USER_OBJECTION"
}
```

**Thread status after:** `escalated`
**User message:** *"Understood — our team will review and correct your payment record."*

---

## Agent 5: Reporting Agent

The Reporting Agent handles financial report generation. It currently has no fallback or task-creation path. A new HITL fallback tool is needed.

---

### Case RR-1: Vendor or contact not resolved for Statement of Account

**Trigger:** The Match Contact tool in the Reporting agent's match vendor sub-workflow loops or reaches a dead end — the user requests a Statement of Account for a specific contact but that contact cannot be matched with sufficient confidence.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Report — contact could not be matched for Statement of Account",
  "description": "The user requested a Statement of Account but the specified contact could not be matched in the accounting system. Manual contact resolution is needed before the report can be generated.",
  "sourceType": "ai_agent",
  "category": "issue",
  "entity": "report",
  "data": {
    "reportType": "statement_of_account",
    "contactNameProvided": "{{contactNameFromUser}}",
    "candidateMatches": [
      { "id": "{{contactId}}", "name": "{{contactName}}", "score": "{{score}}" }
    ],
    "requestedDateRange": {
      "from": "{{fromDate}}",
      "to": "{{toDate}}"
    }
  },
  "errorCode": "REPORT_CONTACT_UNRESOLVED"
}
```

**Thread status after:** `escalated`
**User message:** *"We couldn't locate that contact in your records. Our team will look into it and send you the report."*

---

### Case RR-2: User requests an unsupported or complex report type

**Trigger:** The user asks for a report type or a level of customization that the Reporting agent's Generate Reports and Generate Statement of Account tools do not support (e.g., a custom segmented P&L, a report by cost centre before that feature is live, or a comparative multi-period report).

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Report request — type not supported by agent",
  "description": "The user requested a report that the Reporting agent cannot generate automatically. The request details are preserved for the accountant to prepare manually or advise the user.",
  "sourceType": "ai_agent",
  "category": "normal_task",
  "entity": "report",
  "data": {
    "userReportRequest": "{{verbatimUserMessage}}",
    "detectedReportType": "{{agentClassifiedType}}",
    "requestedDateRange": {
      "from": "{{fromDateOrNull}}",
      "to": "{{toDateOrNull}}"
    },
    "requestedContact": "{{contactOrNull}}"
  },
  "errorCode": "REPORT_TYPE_UNSUPPORTED"
}
```

**Thread status after:** `escalated`
**User message:** *"That report isn't available automatically right now. Our team will prepare it for you."*

---

### Case RR-3: Report generation API failure

**Trigger:** Generate Reports or Generate Statement of Account tool posts to the backend and receives a non-recoverable error from the Wafeq or Zoho reporting API.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Report generation failed — accounting system error",
  "description": "The reporting agent could not generate the requested report due to an API error from the connected accounting system.",
  "sourceType": "ai_agent",
  "category": "error",
  "entity": "report",
  "data": {
    "reportType": "{{reportType}}",
    "requestedDateRange": {
      "from": "{{fromDate}}",
      "to": "{{toDate}}"
    },
    "contactId": "{{contactIdOrNull}}",
    "apiErrorCode": "{{providerErrorCode}}",
    "apiErrorMessage": "{{providerErrorMessage}}"
  },
  "errorCode": "REPORT_API_FAILURE"
}
```

**Thread status after:** `escalated`
**User message:** *"We ran into a technical issue generating your report. Our team will produce it manually and send it to you."*

---

## Agent 6: Onboarding Agent

The Onboarding Agent guides users through connecting Wafeq or Zoho and configuring their accounting setup. It currently has no task-creation path. A new HITL fallback tool is needed.

---

### Case O-1: OAuth connection failed — accounting system could not be linked

**Trigger:** The user clicks the auth URL generated by Get Wafeq Auth Url or Get Zoho Auth Url, completes or attempts the OAuth flow, but the backend does not receive a valid token — the connection is not established.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Accounting system connection failed during onboarding",
  "description": "The user attempted to connect their accounting system via OAuth but the connection was not completed successfully. Manual assistance is needed to complete the setup.",
  "sourceType": "ai_agent",
  "category": "issue",
  "entity": "general_task",
  "data": {
    "accountingProvider": "{{wafeq_or_zoho}}",
    "oauthAttemptTimestamp": "{{timestamp}}",
    "oauthErrorDetail": "{{errorDetailOrNull}}",
    "userOrganizationName": "{{orgName}}"
  },
  "errorCode": "ONBOARDING_OAUTH_FAILED"
}
```

**Thread status after:** `escalated`
**User message:** *"We had trouble connecting your accounting system. Our team will help you complete the setup."*

---

### Case O-2: Accounting system misconfiguration — incorrect tax settings

**Trigger:** After connecting, Pinto reads the user's accounting system configuration and detects that tax settings are not valid — for example, the user has set up custom tax rates in Wafeq or Zoho that conflict with UAE VAT rules (e.g., a rate that is not 0% or 5%, or a tax group applied to the wrong account type).

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "Accounting system tax configuration is invalid",
  "description": "The connected accounting system has tax settings that are incorrect or incompatible with UAE VAT rules. This will cause errors in expense, invoice, and payment workflows if not corrected. Accountant must review and fix the tax configuration before the user can proceed.",
  "sourceType": "ai_agent",
  "category": "issue",
  "entity": "general_task",
  "data": {
    "accountingProvider": "{{wafeq_or_zoho}}",
    "detectedIssues": [
      {
        "field": "{{taxFieldName}}",
        "currentValue": "{{currentValue}}",
        "expectedValue": "{{expectedValue}}",
        "description": "{{issueDescription}}"
      }
    ],
    "userOrganizationName": "{{orgName}}"
  },
  "errorCode": "ONBOARDING_TAX_CONFIG_INVALID"
}
```

**Thread status after:** `escalated`
**User message:** *"We noticed a configuration issue with your accounting system's tax settings. Our team will review and fix this before you start recording transactions."*

---

### Case O-3: Update user info failed — profile setup incomplete

**Trigger:** The Update User Info tool is called as part of onboarding but the backend API call fails, leaving the user profile incomplete and the organization unable to proceed to normal operations.

**Request body:**
```json
{
  "metadata": {
    "threadId": "{{threadId}}",
    "userId": "{{userId}}"
  },
  "title": "User profile setup could not be completed",
  "description": "The onboarding agent could not complete the user profile update step. The organization setup is incomplete and the user cannot yet use Pinto's core features.",
  "sourceType": "ai_agent",
  "category": "error",
  "entity": "general_task",
  "data": {
    "attemptedFields": {
      "businessName": "{{businessNameOrNull}}",
      "vatRegistered": "{{vatRegisteredOrNull}}",
      "trn": "{{trnOrNull}}",
      "businessType": "{{businessTypeOrNull}}"
    },
    "apiErrorMessage": "{{errorMessage}}"
  },
  "errorCode": "ONBOARDING_PROFILE_UPDATE_FAILED"
}
```

**Thread status after:** `escalated`
**User message:** *"We couldn't finish setting up your profile. Our team will complete this for you."*

---

## Summary Table — All Cases

| Case ID | Agent | Category | Entity | errorCode |
|---|---|---|---|---|
| R-1 | Router | `normal_task` | `general_task` | `ROUTER_UNRESOLVABLE_INTENT` |
| R-2 | Router | `error` | `general_task` | `ROUTER_PIPELINE_CRASH` |
| R-3 | Router | `issue` | `general_task` | `ROUTER_TRN_SETUP_FAILED` |
| E-1 | Expense | `issue` | `bill_ocr` | `EXPENSE_OCR_FIELDS_MISSING` |
| E-2 | Expense | `issue` | `bill_ocr` | `EXPENSE_VENDOR_MATCH_FAILED` |
| E-3 | Expense | `issue` | `bill_ocr` | `EXPENSE_TRN_INVALID` |
| E-4 | Expense | `issue` | `bill_manual_entry` | `EXPENSE_ACCOUNT_UNRESOLVED` |
| E-5 | Expense | `error` | `bill_ocr` | `EXPENSE_POST_API_FAILURE` |
| E-6 | Expense | `user_objection` | `bill_ocr` | `EXPENSE_USER_VAT_OBJECTION` |
| E-7 | Expense | `user_objection` | `bill_manual_entry` | `EXPENSE_USER_CATEGORY_OBJECTION` |
| I-1 | Invoice | `issue` | `invoice` | `INVOICE_CUSTOMER_CREATE_FAILED` |
| I-2 | Invoice | `issue` | `invoice` | `INVOICE_ITEM_UNRESOLVED` |
| I-3 | Invoice | `error` | `invoice` | `INVOICE_POST_API_FAILURE` |
| I-4 | Invoice | `normal_task` | `invoice` | `INVOICE_CONVERSATION_TIMEOUT` |
| I-5 | Invoice | `user_objection` | `invoice` | `INVOICE_USER_OBJECTION` |
| P-1 | Payment | `issue` | `payment` | `PAYMENT_CONTACT_UNRESOLVED` |
| P-2 | Payment | `issue` | `payment` | `PAYMENT_INVOICE_NOT_FOUND` |
| P-3 | Payment | `issue` | `payment` | `PAYMENT_BANK_STATEMENT_PARSE_FAILED` |
| P-4 | Payment | `error` | `payment` | `PAYMENT_POST_API_FAILURE` |
| P-5 | Payment | `user_objection` | `payment` | `PAYMENT_USER_OBJECTION` |
| RR-1 | Reporting | `issue` | `report` | `REPORT_CONTACT_UNRESOLVED` |
| RR-2 | Reporting | `normal_task` | `report` | `REPORT_TYPE_UNSUPPORTED` |
| RR-3 | Reporting | `error` | `report` | `REPORT_API_FAILURE` |
| O-1 | Onboarding | `issue` | `general_task` | `ONBOARDING_OAUTH_FAILED` |
| O-2 | Onboarding | `issue` | `general_task` | `ONBOARDING_TAX_CONFIG_INVALID` |
| O-3 | Onboarding | `error` | `general_task` | `ONBOARDING_PROFILE_UPDATE_FAILED` |

---

## Backend Action Required

The `category` enum in `CreateTaskDto` must be updated to include `user_objection`:

```json
"category": {
  "enum": [
    "issue",
    "normal_task",
    "error",
    "user_objection"
  ]
}
```

This change is required before any agent can use cases E-6, E-7, I-5, or P-5.

---

## n8n Implementation Note

Each agent that does not yet have a HITL fallback tool (Invoice, Reporting, Onboarding) should implement one following the pattern of the existing `Fallback dummy` (Expense) and `Payment Fallback` (Payment) workflows:

1. `When Executed by Another Workflow` — receives context payload from the agent
2. `Edit Fields` — assembles the `CreateTaskDto` body from available thread/agent state
3. `Handle creating a task` — HTTP POST to `/ai/organizations/tasks`
4. **On success:** `Update thread status` → `new task were created in web app` → send WhatsApp message to user
5. **On failure:** `Update thread status1` → `Unable to create task` → record fallback state in Postgres for manual follow-up
6. `Insert or update rows in a table` — persist final state to Postgres/Data Tables
