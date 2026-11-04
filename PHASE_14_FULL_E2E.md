# Phase 14 — Full End-to-End: Lead → Payout (The Complete Loop)

> **Concept:** THE ultimate test — one job from start to money in the bank.
> Combines all previous phases into one continuous flow.
> Every step must succeed, and at the end all records must be linked via the `created_from` chain.

**Prerequisite:** Phase 1 complete (system fully configured).

---

## QA Test Case Summary

| User Story | Test Scenario | Test Step | Result | Status |
|---|---|---|---|---|
| User runs 1 complete job from Lead to Payout | TC-14.01 Lead Intake | Create lead: "Emily Park", high urgency, "AC stopped, 95°F, needs repair + possible replacement" | Lead created, status=new, AI summary available |  |
| User runs 1 complete job from Lead to Payout | TC-14.02 Convert → Quote → Send | Convert lead to quote, add 4 lines (AC Unit $2,500, Labor $450, Trip Fee $75, Loyalty Discount -$50 optional), set 50% deposit, send | Customer created, quote sent with portal link, deposit=$1,512.50 |  |
| User runs 1 complete job from Lead to Payout | TC-14.03 Customer Approve + Deposit | Customer opens portal, ticks optional discount (total=$2,975), signs, pays deposit ($1,487.50) | Quote approved+locked, deposit invoice+payment auto-created, WO auto-created from quote |  |
| User runs 1 complete job from Lead to Payout | TC-14.04 Dispatch Tech | Open WO, assign tech, schedule visit today | Visit on dispatch board in assigned tech's row |  |
| User runs 1 complete job from Lead to Payout | TC-14.05 Tech Starts Job | Tech opens My Day, taps job, starts visit | Visit in_progress, timer auto-starts |  |
| User runs 1 complete job from Lead to Payout | TC-14.06 Complete Checklist | Complete photo, reading, signature (pre-work), install gate items (serial SN-2024-PARK-001, nameplate photo) | All required items done, "0 required open" |  |
| User runs 1 complete job from Lead to Payout | TC-14.07 Photos | Upload before + after photos | Photos visible in Before/After grid sections |  |
| User runs 1 complete job from Lead to Payout | TC-14.08 Record Expense | Add expense: Refrigerant R-410A $50, billable, submit | Expense created and submitted |  |
| User runs 1 complete job from Lead to Payout | TC-14.09 Checkout + Pay | Stop timer, open checkout (balance=$1,487.50 after deposit), sign-off docs, customer signature, collect payment, complete job | WO complete, equipment auto-created (AC Unit, serial SN-2024-PARK-001), visit completed |  |
| User runs 1 complete job from Lead to Payout | TC-14.10 Admin Approve Expense | Find submitted $50 expense, approve | Cost posted to WO |  |
| User runs 1 complete job from Lead to Payout | TC-14.11 Create Invoice | Create invoice from completed WO | Invoice draft with WO lines + billable expense (with markup if configured) |  |
| User runs 1 complete job from Lead to Payout | TC-14.12 Send Invoice | Review and send invoice | Status=open, locked, customer notified |  |
| User runs 1 complete job from Lead to Payout | TC-14.13 Customer Pays Balance | Customer opens portal, pays remaining balance | Invoice status=paid, balance=$0 |  |
| User runs 1 complete job from Lead to Payout | TC-14.14 Payout | Open Payments → Payouts tab | Payout containing this job's charges: gross, fee, net (gross − fees = bank amount), charges mapped to customer payments |  |
| User runs 1 complete job from Lead to Payout | TC-14.15 Verify created_from Chain | Navigate through all records | Lead → convertedQuoteKey → Quote → WO (created_from=Q_), WO → Invoice (workOrderId), Invoice → Payment, Equipment → sourceWoId, Payout → charges. All bidirectional links verified |  |

**Reference:** [StrongHeron QA Test Cases.md](../../StrongHeron%20QA%20Test%20Cases.md) — Phase 14

---

## Step 14.01 — Lead Intake

**Test Scenario:** Step 14.01 — Lead Intake

**App:** Admin → Leads

| Test Step | Result |
|--------|----------|
| New lead appears (or create manually) | Status = `new` |
| AI summary available | Parsed from source |

**Test data:**

| Field | Value |
|-------|-------|
| Name | "Emily Park" |
| Phone | "+15125559999" |
| Email | "emily@park.com" |
| Source | "Phone" |
| Urgency | "high" |
| Summary | "AC completely stopped, house is 95°F, needs emergency repair + possible unit replacement" |

---

## Step 14.02 — Convert Lead → Quote → Send

**Test Scenario:** Step 14.02 — Convert Lead → Quote → Send

**App:** Admin → Lead Record → Quote

| Test Step | Result |
|--------|----------|
| Convert lead → "Create quote" | Customer created, quote builder opens |
| Add line 1: AC Unit ($2,500, material, creates_equipment) | Line 1 |
| Add line 2: Installation Labor ($450, Preferred price) | Line 2 |
| Add line 3: Trip Fee ($75, fee) | Line 3 |
| Add line 4: Loyalty Discount (-$50, optional) | Line 4 (optional) |
| Set deposit: 50% required | Deposit = $1,512.50 |
| Send quote | Status = `sent`, customer notified |

**Chain so far:** Lead → Quote

---

## Step 14.03 — Customer Approves + Pays Deposit

**Test Scenario:** Step 14.03 — Customer Approves + Pays Deposit

**App:** Quote Portal (`/p/:token`)

| Test Step | Result |
|--------|----------|
| Customer opens portal | Quote details visible |
| Tick optional Loyalty Discount | Total recalculates to $2,975 |
| Sign approval | Signature captured |
| Pay deposit ($1,487.50) via card | Payment processed |
| Quote status = `approved`, locked | Cannot edit |
| **Deposit Invoice auto-created** | INV with deposit amount |
| **Deposit Payment auto-created** | Payment linked to deposit invoice |
| **WO auto-created from quote** | Lines carried over |

**Chain so far:** Lead → Quote → Deposit Invoice + Payment → WO

---

## Step 14.04 — Dispatch Tech

**Test Scenario:** Step 14.04 — Dispatch Tech

**App:** Admin → WO Record

| Test Step | Result |
|--------|----------|
| Open WO created from approved quote | WO record visible |
| Assign tech + schedule visit (today) | Visit created |
| Visit on dispatch board | Row for assigned tech |

---

## Step 14.05 — Tech Starts Job

**Test Scenario:** Step 14.05 — Tech Starts Job

**App:** PWA → My Day → Job

| Test Step | Result |
|--------|----------|
| Tech opens My Day | Visit card visible |
| Tap job → "Start visit" | Visit status → `in_progress` |
| Timer auto-starts | Time tracking begins |
| Navigate to time screen | Timer UI visible |

---

## Step 14.06 — Tech Completes Checklist

**Test Scenario:** Step 14.06 — Tech Completes Checklist

**App:** PWA → Checklist

| Test Step | Result |
|--------|----------|
| Open checklist | Items from job type template |
| Complete photo item | Upload photo → saved |
| Complete reading item | Enter value → saved |
| Complete signature item (pre-work) | Signed → saved |
| Install gate items (AC Unit has creates_equipment): | |
| — Enter serial: "SN-2024-PARK-001" | `install_serial` saved |
| — Upload nameplate photo | `install_nameplate_photo` saved |
| All required items done | "0 required open" |

---

## Step 14.07 — Tech Takes Photos

**Test Scenario:** Step 14.07 — Tech Takes Photos

**App:** PWA → Photos

| Test Step | Result |
|--------|----------|
| Upload before photo | `kind: 'before_photo'` |
| Upload after photo | `kind: 'after_photo'` |
| Photos visible in grid | Before (1), After (1) |

---

## Step 14.08 — Tech Records Expense

**Test Scenario:** Step 14.08 — Tech Records Expense

**App:** PWA → Expenses

| Test Step | Result |
|--------|----------|
| Add expense: "Refrigerant R-410A" | Category, vendor, amount |
| Amount: $50.00, billable: checked | Billable expense |
| Submit expense | Status → `submitted` |

---

## Step 14.09 — Tech Checkout + Pay

**Test Scenario:** Step 14.09 — Tech Checkout + Pay

**App:** PWA → Checkout

| Test Step | Result |
|--------|----------|
| Stop timer → time entry created | Duration tracked |
| Open checkout | Totals breakdown: |
| — Subtotal: $2,975.00 | From quote lines |
| — Deposit paid: -$1,487.50 | From Step 14.03 |
| — **Balance due: $1,487.50** | Remaining |
| Sign-off checkout documents | Documents signed |
| Customer signature | Captured |
| Collect payment → Card | `recordManualPayment()` or card charge |
| **Complete job** | `POST /work-orders/:id/complete` |
| Done screen | Summary visible |

**On complete (server-side):**
- WO status → `complete`
- Equipment auto-created (AC Unit with serial SN-2024-PARK-001)
- Visit status → completed

---

## Step 14.10 — Admin Approves Expense

**Test Scenario:** Step 14.10 — Admin Approves Expense

**App:** Admin → Expenses

| Test Step | Result |
|--------|----------|
| Find submitted expense ($50 R-410A) | Listed in pending |
| Approve | `approveExpense(id)` |
| Cost posted to WO | WO actual cost updated |

---

## Step 14.11 — Admin Creates Invoice

**Test Scenario:** Step 14.11 — Admin Creates Invoice

**App:** Admin → WO Record

| Test Step | Result |
|--------|----------|
| WO (completed) → "Create invoice" | Invoice draft created |
| Lines from WO + approved billable expense line | All lines present |
| Expense billed with markup (if configured) | Amount × (1 + markup%) |

**Invoice lines should include:**

| Line | Source | Amount |
|------|--------|--------|
| AC Unit | WO line (from quote) | $2,500.00 |
| Installation Labor | WO line (from quote) | $450.00 |
| Trip Fee | WO line (from quote) | $75.00 |
| Loyalty Discount | WO line (optional, ticked) | -$50.00 |
| Refrigerant R-410A | Billable expense | $50.00 (+markup) |

---

## Step 14.12 — Send Invoice

**Test Scenario:** Step 14.12 — Send Invoice

**App:** Admin → Invoice Record

| Test Step | Result |
|--------|----------|
| Review lines + totals | Math correct |
| Send invoice | Status = `open`, locked |
| Customer notified | Email/SMS with portal link |

---

## Step 14.13 — Customer Pays Remaining Balance

**Test Scenario:** Step 14.13 — Customer Pays Remaining Balance

**App:** Invoice Portal

| Test Step | Result |
|--------|----------|
| Customer opens invoice portal | Balance visible |
| Pay remaining balance | Card charged |
| Invoice status = `paid` | Fully paid |
| Balance = $0.00 | Zero |

---

## Step 14.14 — Payout Arrives

**Test Scenario:** Step 14.14 — Payout Arrives

**App:** Admin → Payments → Payouts tab

**Payments page tabs:**

| Tab | Content |
|-----|---------|
| Payments | All payment records (pending, succeeded, failed, refunded, disputed) |
| Payouts | Stripe payouts to bank account |
| Disputes | Chargebacks filed via Stripe webhooks |
| Approvals | Pending payment requests |

**Payout detail (drilldown):**

| Field | Description |
|-------|-------------|
| Payout ID | Stripe payout ID (monospace) |
| Gross | Total charges |
| Fee | Stripe processing fees |
| Net | Gross - Fee = amount to bank |
| Status | pending / paid / failed |
| Charges | Individual charge records mapped to this payout |

| Test Step | Result |
|--------|----------|
| Open Payments → Payouts tab | Payout list visible |
| Find payout containing this job's charges | Payout record |
| Drilldown → verify gross, fee, net | Math: gross − fees = net |
| Charges map to this customer's payments exactly | Traceable |

---

## Step 14.15 — Verify `created_from` Chain

**Test Scenario:** Step 14.15 — Verify `created_from` Chain

**App:** Admin → navigate through all records

**Complete transform chain:**
```
Lead (new → contacted → converted)
  ↓ convertLead({ createQuote: true })
Customer (created from lead data)
  ↓
Quote (draft → sent → approved → locked)
  ↓ quote approval
Deposit Invoice (draft → paid)  +  Deposit Payment
  ↓ create WO from approved quote
Work Order (new → scheduled → in_progress → complete)
  ↓ equipment birth on complete
Equipment (created from WO, serial captured)
  ↓ create invoice from WO
Invoice (draft → open → paid)
  ↓ customer payment
Payment (succeeded)
  ↓ Stripe processing
Payout (to bank)
```

**Verification checklist:**

| Record | Link | Verified |
|--------|------|----------|
| Lead → `convertedQuoteKey` | Points to Quote | ☐ |
| Lead → customer created | Customer exists | ☐ |
| Quote → `created_from` = Lead | Source link | ☐ |
| Quote → Related → WO link | Bidirectional | ☐ |
| WO → `created_from` = Quote key | Source link | ☐ |
| WO → Related → Quote link | Bidirectional | ☐ |
| WO → customer = Quote customer | Same customer | ☐ |
| Invoice → `workOrderId` = WO | Source WO link | ☐ |
| Invoice → customer = WO customer | Same customer | ☐ |
| Payment → linked to Invoice | Payment → Invoice | ☐ |
| Equipment → source WO | Install WO link | ☐ |
| Equipment → customer + site | Correct assignment | ☐ |
| Payout → contains payment charges | Charge mapping | ☐ |

---

## ✅ Phase 14 Results — THE COMPLETE LOOP

| Item | Status |
|------|--------|
| Lead intake | ☐ |
| Convert → Quote → send | ☐ |
| Customer approves + pays deposit | ☐ |
| Deposit Invoice + Payment auto-created | ☐ |
| WO auto-created from quote | ☐ |
| Dispatch tech | ☐ |
| Tech starts job + timer | ☐ |
| Checklist complete (incl. install gates) | ☐ |
| Photos (before/after) | ☐ |
| Expense recorded + submitted | ☐ |
| Checkout + payment collected | ☐ |
| Job complete → equipment birthed | ☐ |
| Admin approves expense | ☐ |
| Create invoice (WO lines + expense) | ☐ |
| Send invoice | ☐ |
| Customer pays remaining | ☐ |
| Payout to bank | ☐ |
| **Full `created_from` chain verified** | ☐ |

**The complete loop: Lead → Customer → Quote → Deposit → WO → Tech job → Equipment → Checkout → Payment → Invoice → Customer pay → Payout to bank. All linked, all traceable.**

---

## All 14 Phases Complete

| Phase | Test File | Steps |
|-------|-----------|:-----:|
| 1 | PHASE_1_SETUP_ONBOARDING.md | 15 |
| 2 | PHASE_2_LEAD_TO_WO.md | 9 |
| 3 | PHASE_3_LEAD_TO_QUOTE_TO_WO.md | 15 |
| 4 | PHASE_4_TECH_JOB_EXECUTION.md | 15 |
| 5 | PHASE_5_CHECKOUT_PAYMENT.md | 10 |
| 6 | PHASE_6_DEPOSIT_NTE_GATE.md | 10 |
| 7 | PHASE_7_EQUIPMENT_INSTALL.md | 10 |
| 8 | PHASE_8_INVOICE_LIFECYCLE.md | 13 |
| 9 | PHASE_9_SERVICE_PLAN.md | 13 |
| 10 | PHASE_10_PROJECT_MULTIPHASE.md | 14 |
| 11 | PHASE_11_EXPENSES_FLOW.md | 8 |
| 12 | PHASE_12_TIME_ENTRY_FLOW.md | 11 |
| 13 | PHASE_13_OWNER_MOBILE.md | 8 |
| 14 | PHASE_14_FULL_E2E.md | 15 |
| **TOTAL** | | **166** |
