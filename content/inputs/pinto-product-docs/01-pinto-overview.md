---
title: "01-pinto-overview"
publish: true
---

# Pinto – Product Overview

## What is Pinto?

Pinto is an AI-powered accounting assistant designed to simplify financial operations for micro-businesses, primarily in the UAE with plans to expand across the GCC and MENA region. It operates through WhatsApp as its primary channel, turning complex accounting tasks into simple, conversational interactions — no accounting expertise required.

At its core, Pinto acts as a smart layer between businesses and their accounting software. It combines AI intelligence, document automation, and human oversight to handle expenses, invoices, payments, financial reporting, and VAT compliance — all for a fraction of the cost of a traditional accountant.

---

## The Problem Pinto Solves

Micro-businesses in the UAE face a set of persistent, costly financial challenges:

- Hiring a qualified bookkeeper costs between $500–$1,500 per month
- Sorting and categorizing receipts manually takes 5–8 hours per week
- VAT compliance is complex, with fear of FTA penalties being widespread
- Creating and tracking invoices is time-consuming without dedicated tools
- Business owners have no real-time view of their financial health
- Bank reconciliation and chasing unpaid invoices create ongoing cash flow stress

Pinto was built specifically to eliminate these pain points.

---

## How Pinto Works

The user experience is deliberately simple:

1. A business owner sends a message on WhatsApp — a photo of a receipt, a request to create an invoice, or a question like "What's my profit this month?"
2. Pinto's AI understands the request, asks clarifying questions if needed, and completes the task automatically
3. The accounting system (Wafeq or Zoho Books) is updated in real time
4. If the AI cannot resolve something with confidence, the case is escalated to a human accountant for review

---

## The Three Parts of Pinto

Pinto is composed of three interconnected components, each serving a distinct role:

**1. The Pinto Agent** is the conversational AI engine that lives on WhatsApp. It processes natural language, reads documents via OCR, categorizes transactions, creates entries in the accounting system, and handles the majority of financial tasks automatically. It is the primary interface for business owners.

**2. The Pinto App** is a web application for business users. It provides a dashboard showing open tasks (issues that need the user's attention), invoice management, expense tracking, and a system for responding to inquiries sent by accountants. Users log in via WhatsApp OTP and can resolve flagged issues directly through the app.

**3. The Accountant Dashboard** is a Retool-based internal tool used by Pinto's accountant team. It receives tasks escalated by the AI agent, allows accountants to review and fix financial entries, and enables them to send inquiries back to users when clarification is needed. It is the human-in-the-loop layer that ensures accuracy and quality.

---

## Core Capabilities

**Expense Management** — Users photograph receipts and Pinto extracts the amount, vendor, VAT, and category automatically. Bulk uploads are supported. TRN validation is performed against the FTA database.

**Invoice & Billing** — Invoices are created through conversation. Pinto handles customer matching, VAT calculation, PDF generation, and WhatsApp delivery to clients.

**Payment Tracking** — Payments are recorded against invoices, including partial payments, overpayments (stored as advance credit), and bulk payment allocation.

**Financial Reporting** — Users can request Profit & Loss statements, balance sheets, cash flow reports, and statements of account at any time via WhatsApp or the web dashboard.

**VAT Compliance** — All transactions are VAT-aware. Pinto automatically applies 5% UAE VAT, distinguishes standard from zero-rated transactions, validates TRNs, and maintains audit-ready records.

**Accounting System Integration** — Pinto integrates with Wafeq and Zoho Books via secure OAuth, syncing every action in real time without requiring users to interact with the accounting software directly.

---

## Technology Stack (High Level)

- **WhatsApp Business API (WATI)** — Primary user communication channel
- **n8n** — Automation workflow orchestration
- **AWS (ECS / RDS)** — Backend infrastructure
- **BigQuery** — Analytics and data warehouse
- **Retool** — Accountant dashboard
- **Wafeq / Zoho Books** — Accounting system integrations

---

## Vision

Pinto aims to become the AI financial operating layer for businesses across the MENA region — transforming accounting from a manual, expensive process into a conversational, intelligent experience accessible to every business owner, regardless of their financial background.
