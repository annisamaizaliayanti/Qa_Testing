# Phase 8 — Invoice Flow: Create, Send, Customer Pays, Credit Memo

> **Concept:** Full invoice lifecycle — from completed WO, create invoice, send to customer, customer pays partially via portal, pays remainder, handle overdue, credit memo, refund.
> Test all status transitions: draft → open → partially_paid → paid, plus overdue, credit memo, and refund.

**Prerequisite:** Phase 5 complete (completed WO with payment exists). Or create and complete a new WO.

---

## QA Test Case Summary

| User Story | Test Scenario | Test Step | Result | Status |
|---|---|---|---|---|
| Admin manages invoice lifecycle from draft to paid/refund | TC-8.01 Create Invoice from WO | Open completed WO, click "Create invoice" | Invoice draft created with WO lines copied, created_from=WO key |  |
| Admin manages invoice lifecycle from draft to paid/refund | TC-8.02 Review Invoice | Open invoice record | Customer, issue/due dates, status=draft, lines with correct subtotal/tax/total, editable while draft |  |
| Admin manages invoice lifecycle from draft to paid/refund | TC-8.03 Send Invoice | Click "Send" (email or SMS) | Status=open, locked_at set (no more edits), customer notified with portal link |  |
| Admin manages invoice lifecycle from draft to paid/refund | TC-8.04 Customer Portal View | Customer opens /p/inv/:token | Invoice lines, totals, due date visible; pay button available; firstViewedAt recorded |  |
| Admin manages invoice lifecycle from draft to paid/refund | TC-8.05 Partial Payment | Customer pays $500 of $3,000 total | Status=partially_paid, paidCents=$500, balanceCents=$2,500 |  |
| Admin manages invoice lifecycle from draft to paid/refund | TC-8.06 Admin Verify Partial | Admin opens same invoice | Status badge=partially_paid, Paid=$500 (green), Balance=$2,500, payment history linked |  |
| Admin manages invoice lifecycle from draft to paid/refund | TC-8.07 Full Payment | Customer pays remaining $2,500 | Status=paid, balance=$0 |  |
| Admin manages invoice lifecycle from draft to paid/refund | TC-8.08 Overdue Invoice | Create new invoice with due date 15 days in the past, send it | Status=open, overdue flag appears (dueDate < today && status=open), days computed in frontend |  |
| Admin manages invoice lifecycle from draft to paid/refund | TC-8.09 Send Reminder | Click "Remind now" on overdue invoice | Reminder sent to customer (email/SMS), logged in history |  |
| Admin manages invoice lifecycle from draft to paid/refund | TC-8.10 Credit Memo | Create credit memo: amount=$200, reason="billing_error" | CM created with CM- key prefix, status after issue=open |  |
| Admin manages invoice lifecycle from draft to paid/refund | TC-8.11 Apply Credit Memo | Apply CM ($200) to an open invoice | Invoice balance decreases by $200, CM status=applied |  |
| Admin manages invoice lifecycle from draft to paid/refund | TC-8.12 Refund | Refund $100 from a payment | Payment status=partially_refunded, refunded=true, refund record created |  |
| Admin manages invoice lifecycle from draft to paid/refund | TC-8.13 Refund Cap (Step-up) | Tech role attempts refund above role cap | 409 STEP_UP_REQUIRED (applies only to custom roles with refundCapCents > 0; system roles have no cap) |  |

**Reference:** [StrongHeron QA Test Cases.md](../../StrongHeron%20QA%20Test%20Cases.md) — Phase 8

---

## Step 8.01 — Create Invoice from WO

**Test Scenario:** Step 8.01 — Create Invoice from WO

**App:** Admin → WO Record (completed)

| Test Step | Result |
|--------|----------|
| Open completed WO | WO record displayed |
| Click "Create invoice" | `createInvoiceFromWo()` called |
| Invoice draft created | Lines copied from WO |
| Navigate to invoice record | Invoice page displayed |

---

## Step 8.02 — Review Invoice Lines

**Test Scenario:** Step 8.02 — Review Invoice Lines

**App:** Admin → Invoice Record (`/invoices/:id`)

**Invoice fields displayed:**

| Field | Value |
|-------|-------|
| Customer | Auto from WO |
| Issue date | Today (default) |
| Due date | Based on payment terms |
| Customer PO | From WO if exists |
| Status | `draft` |
| Work order | Link to source WO |

**Totals section:**

| Label | Display |
|-------|---------|
| Subtotal | `invoice.subtotalCents` |
| Tax | `invoice.taxCents` |
| **Total** | `invoice.totalCents` (bold) |
| Paid | `invoice.paidCents` (green) |
| **Balance** | Total - Paid |

**Line items (from WO):**

| Column | Description |
|--------|-------------|
| Item | Select from catalog |
| Description | Text override |
| Qty | Number |
| Rate | Number (dollars) |
| Total | Computed |

| Test Step | Result |
|--------|----------|
| Review lines | Math correct (subtotal, tax, total) |
| Edit line if needed (still draft) | Editable |

---

## Step 8.03 — Send Invoice

**Test Scenario:** Step 8.03 — Send Invoice

**App:** Admin → Invoice Record

**Send options:**

| Field | Type | Options |
|-------|------|---------|
| Channel | select | email / sms |
| Recipient | input | Email address or phone number |

| Test Step | Result |
|--------|----------|
| Click "Send" | Invoice status = `open` |
| Invoice locked | `locked_at` present, lines cannot be edited |
| Customer notified | Email/SMS sent with portal link |

---

## Step 8.04 — Customer Opens Invoice Portal

**Test Scenario:** Step 8.04 — Customer Opens Invoice Portal

**App:** Invoice Portal (`/p/inv/:token`)

| Test Step | Result |
|--------|----------|
| Customer opens link from email/SMS | Invoice portal displayed |
| Lines, totals, due date visible | All data correct |
| Pay button available | Customer can pay |
| `firstViewedAt` recorded | View tracking |

---

## Step 8.05 — Customer Pays Partially

**Test Scenario:** Step 8.05 — Customer Pays Partially

**App:** Invoice Portal

| Test Step | Result |
|--------|----------|
| Customer pays $500 of $3,000 total | Payment processed |
| Invoice status = `partially_paid` | Status changes |
| `paidCents` = 50000 | $500 recorded |
| `balanceCents` = $2,500 | Remaining balance |

---

## Step 8.06 — Check in Admin

**Test Scenario:** Step 8.06 — Check in Admin

**App:** Admin → Invoice Record

| Test Step | Result |
|--------|----------|
| Open same invoice | Record updated |
| Status badge = `partially_paid` | Visual indicator |
| Paid = $500.00 (green) | Amount shown |
| Balance = $2,500.00 | Remaining |
| Payment history visible | Payment record linked |

---

## Step 8.07 — Customer Pays Remainder

**Test Scenario:** Step 8.07 — Customer Pays Remainder

**App:** Invoice Portal

| Test Step | Result |
|--------|----------|
| Customer pays remaining $2,500 | Payment processed |
| Invoice status = `paid` | Fully paid |
| Balance = $0.00 | Zero balance |

---

## Step 8.08 — Overdue Scenario

**Test Scenario:** Step 8.08 — Overdue Scenario

**App:** Admin → Invoice New (`/invoices/new`)

**InvoiceNew form fields:**

| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| Customer | select | ✅ | Select customer |
| Issue date | date | - | 30 days ago |
| Due date | date | - | 15 days ago (already past) |
| Customer PO | input text | - | "PO-LATE-001" |
| Tax rate (%) | number | - | "8.25" |
| Memo | textarea | - | "Overdue test invoice" |

**Line items (repeatable):**

| Field | Type | Example |
|-------|------|--------|
| Item | select | Select from catalog |
| Description | input text | Override description |
| Qty | number | "1" |
| Rate | number | "500.00" |

| Test Step | Result |
|--------|----------|
| Create new invoice → set due date in the past | Invoice draft created |
| Send invoice | Status = `open` |
| Because due date has passed | Overdue flag appears |
| `days_overdue` calculated | Counter displayed |

---

## Step 8.09 — Send Reminder

**Test Scenario:** Step 8.09 — Send Reminder

**App:** Admin → Invoice Record (overdue)

| Test Step | Result |
|--------|----------|
| Click "Remind now" on overdue invoice | Reminder sent to customer |
| Channel (email/sms) selected | Same send options |
| Reminder logged | History updated |

---

## Step 8.10 — Credit Memo (Correction)

**Test Scenario:** Step 8.10 — Credit Memo (Correction)

**App:** Admin → Invoice Record

| Test Step | Result |
|--------|----------|
| Invoice → "Credit" action | Credit memo creation flow |
| Set amount = $200 | Partial credit |
| Reason = "Overcharged labor" | Reason recorded |
| Submit | Credit memo created with `CM-` key prefix |
| CM status = `issued` | Ready to apply |

---

## Step 8.11 — Apply Credit Memo to Invoice

**Test Scenario:** Step 8.11 — Apply Credit Memo to Invoice

**App:** Admin → Invoice Record (another open invoice)

**Apply Credit Memo modal:**

| Field | Type | Description |
|-------|------|-------------|
| SELECT CREDIT MEMO | select | List available CMs for this customer |

| Test Step | Result |
|--------|----------|
| Click "Apply credit memo" | Modal opens |
| `listCreditMemos({ customerId, status: 'issued' })` | Available CMs loaded |
| Select CM ($200) → Apply | `applyCreditMemo(cmId, { invoiceId })` |
| Invoice balance decreases by $200 | Balance updated |
| CM status → applied | CM consumed |

---

## Step 8.12 — Refund to Customer

**Test Scenario:** Step 8.12 — Refund to Customer

**App:** Admin → Payments

| Test Step | Result |
|--------|----------|
| Refund remaining credit to customer card | Refund processed |
| Refund record created | Payment type = refund |
| Customer notified | Confirmation sent |

---

## Step 8.13 — Refund Above Cap → Step-up Required

**Test Scenario:** Step 8.13 — Refund Above Cap → Step-up Required

**App:** Admin (logged in as tech role)

| Test Step | Result |
|--------|----------|
| Try refund $1,000 (tech role, cap=$100) | **409 STEP_UP_REQUIRED** |
| Error message: admin approval required | Role permission gate |
| Login as admin → approve or do refund | Refund succeeds |

---

## ✅ Phase 8 Results

| Item | Status | Notes |
|------|--------|-------|
| Create invoice from WO | ✅ | `POST /work-orders/:id/create-invoice` → INV-1003, status `draft`, $3,000 |
| Send invoice → status open | ✅ | `POST /invoices/:id/send` → status `open`, email sent |
| Partial payment $500 → partially_paid | ✅ | `POST /payments/manual` with `invoiceId` → paidCents $500, status `partially_paid` |
| Full payment remaining $2,500 → paid | ✅ | Second payment → status `paid`, balanceCents $0 |
| Overdue invoice | ✅ | Create invoice with `dueDate` 15 days ago → send → status `open` (overdue by date) |
| Overdue: field `daysOverdue` | — | Not in backend — computed in frontend admin only |
| Send reminder | ✅ | `POST /invoices/:id/send` again on already-open invoice |
| Credit memo CM-0001 | ✅ | `POST /credit-memos` with `reason: 'billing_error'` + lines → CM-0001, $200 |
| Issue credit memo | ✅ | `POST /credit-memos/:id/issue` → status `open` |
| Apply CM to invoice | ✅ | `POST /credit-memos/:id/apply` with `{invoiceId, amountCents}` → CM `applied`, invoice balance reduced |
| Refund $100 from payment | ✅ | `POST /payments/:id/refund` → `partially_refunded`, `refunded: true` |
| Refund cap → step-up required | — | Step-up only applies to custom roles (`!isSystem`). Tech is a system role → no cap. Scenario requires a separate custom role. |

**Full invoice lifecycle: draft → open → partially_paid → paid. Overdue + reminder. Credit memo + apply. Refund.**

### Bugs & Notes

| Finding | Detail |
|--------|--------|
| Endpoint to create invoice from WO | Not `POST /work-orders/:id/invoice` — correct: `POST /work-orders/:id/create-invoice` |
| Credit memo reason | Must be enum: `'goodwill' \| 'service_issue' \| 'billing_error' \| 'return' \| 'overpayment' \| 'other'` |
| Credit memo issue → status `open` | Not `issued` — status after issue is `open`, not `issued` as documented |
| Apply CM requires `amountCents` | `POST /credit-memos/:id/apply` must send `{invoiceId, amountCents}` |
| `daysOverdue` not in backend | Field is not calculated in API — only in frontend. Invoice is considered overdue if `dueDate < today && status = 'open'` |
| Step-up refund | Only applies to **custom roles** with `refundCapCents > 0`. System roles (admin, technician, etc.) have no cap → no step-up |

### Phase 8 Endpoint Flow

```
POST /work-orders/:id/create-invoice          → INV draft
POST /invoices/:id/send                        → INV open (+ lock)
POST /payments/manual {invoiceId, amountCents} → partially_paid / paid
POST /invoices/:id/send (resend)               → reminder
POST /credit-memos {reason, lines}             → CM draft
POST /credit-memos/:id/issue                   → CM open
POST /credit-memos/:id/apply {invoiceId, amountCents} → CM applied, invoice balance reduced
POST /payments/:id/refund {amountCents, reason} → partially_refunded / refunded
```
