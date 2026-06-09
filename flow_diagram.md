# 🏥 Medical Appointment System — Flow Diagram

## System Overview

```
Patient (WhatsApp)
       ↓
   AI Agent (n8n)
       ↓
  Google Sheets
       ↓
  Stripe Payment
       ↓
  Confirmation
```

---

## Workflow 01 — AI Agent Flow

```
Patient sends message (WhatsApp)
       ↓
AI Agent receives message
       ↓
       ├── Book Appointment?
       │       ↓
       │   Check Patient List (Google Sheets)
       │       ↓
       │   Add Appointment (Google Sheets)
       │       ↓
       │   Send Confirmation Message
       │
       ├── Reschedule?
       │       ↓
       │   Update Appointment (Google Sheets)
       │       ↓
       │   Send New Schedule
       │
       └── Cancel?
               ↓
           Cancel Appointment (Google Sheets)
               ↓
           Send Cancellation Confirmation
```

---

## Workflow 02 — Stripe Payment Flow

```
Google Sheets Trigger (new row added)
       ↓
IF: payment_method == "Stripe"?
       ↓
      YES
       ↓
Stripe Checkout Session Generate
POST → https://api.stripe.com/v1/checkout/sessions
       ↓
Get checkout URL ($json.url)
       ↓
Send Payment Link (WhatsApp)
```

---

## Workflow 03 — Payment Verification Flow

```
Stripe Webhook Trigger
(event: checkout.session.completed)
       ↓
GET Checkout Session Details
GET → https://api.stripe.com/v1/checkout/sessions/{id}
       ↓
Get Patient Row (Google Sheets)
       ↓
       ├── Payment SUCCESS?
       │       ↓
       │   Update Sheet → payment_status: "Paid"
       │       ↓
       │   Send Confirmation (WhatsApp)
       │
       └── Payment FAILED?
               ↓
           Send Failure Message (WhatsApp)
```

---

## Workflow 04 — Refund Flow

```
Google Sheets Trigger (row updated)
       ↓
IF1: status == "Cancelled"?
       ↓
IF2: payment_status == "Paid"?
       ↓
      YES
       ↓
Stripe Refund Generate
POST → https://api.stripe.com/v1/refunds
       ↓
Update Sheet → payment_status: "Refunded"
       ↓
Send Refund Confirmation (WhatsApp)
```

---

## Data Flow — Google Sheets Structure

```
Columns:
appointment_id → patient_id → date → time
      ↓               ↓
payment_method   whatsapp_number
      ↓
payment_status (Pending → Paid → Refunded)
      ↓
status (Confirmed → Cancelled)
      ↓
stripe_payment_details
```

---

## Tech Stack Diagram

```
┌─────────────────────────────────────┐
│           Patient Layer             │
│         WhatsApp / Telegram         │
└─────────────────┬───────────────────┘
                  ↓
┌─────────────────────────────────────┐
│         Automation Layer            │
│              n8n                    │
│  ┌──────────┐    ┌───────────────┐  │
│  │ AI Agent │    │ HTTP Request  │  │
│  │  (GPT-4o)│    │  (Stripe API) │  │
│  └──────────┘    └───────────────┘  │
└──────┬──────────────────┬───────────┘
       ↓                  ↓
┌──────────────┐  ┌───────────────────┐
│ Google Sheets│  │   Stripe API      │
│  (Database)  │  │ (Payment Gateway) │
└──────────────┘  └───────────────────┘
```
