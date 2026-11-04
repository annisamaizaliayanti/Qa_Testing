# Phase 3 — Lead Intake → Quote → Portal Approval → Work Order

> **Concept:** The lead requires a quote before agreeing. Admin converts the lead to a Quote, builds line items, and sends it to the customer.
> The customer opens the portal, reviews, selects optional add-ons, signs, and pays a deposit.
> Quote approved → admin creates WO from quote → assigns tech.

**Prerequisite:** Phase 1 complete. Items catalog exists (AC Unit, Installation Labor, Trip Fee, Loyalty Discount).

---

## QA Test Case Summary

| User Story | Test Scenario | Test Step | Result | Status |
|---|---|---|---|---|
| User converts lead to quote, customer approves via portal, then becomes WO | TC-3.01 New Lead for Quote | Create lead via NewLeadModal ("David Chen", wants quote first) | Lead created with status=new |  |
| User converts lead to quote, customer approves via portal, then becomes WO | TC-3.02 Convert Lead → Quote | Click "Create quote" on lead record | Quote builder opens with customer prefilled from lead data, lead status=converted, convertedQuoteKey visible |  |
| User converts lead to quote, customer approves via portal, then becomes WO | TC-3.03 Build Quote Lines | Add AC Unit ($2,500 qty 1), Installation Labor ($450 Preferred price), Trip Fee ($75) | Subtotal = $3,025.00, all lines visible with correct rates |  |
| User converts lead to quote, customer approves via portal, then becomes WO | TC-3.04 Optional Add-on | Add Loyalty Discount (-$50), check "Optional" checkbox | Line marked optional, total remains $3,025.00 (excluded from total) |  |
| User converts lead to quote, customer approves via portal, then becomes WO | TC-3.05 Set Deposit Required | Enable deposit required, set 50% | Deposit amount = $1,512.50 (50% of $3,025) shown in totals |  |
| User converts lead to quote, customer approves via portal, then becomes WO | TC-3.06 Send Quote | Click "Send" in quote header | Status=sent, portal link /p/:token active, customer notified via email/SMS |  |
| User converts lead to quote, customer approves via portal, then becomes WO | TC-3.07 Customer Opens Portal | Customer opens portal link | Approval page shows line items, optional add-on with checkbox, total $3,025.00, deposit $1,512.50 |  |
| User converts lead to quote, customer approves via portal, then becomes WO | TC-3.08 Tick Optional Add-on | Customer checks Loyalty Discount (-$50) | Total recalculates to $2,975.00, deposit recalculates to $1,487.50 |  |
| User converts lead to quote, customer approves via portal, then becomes WO | TC-3.09 Customer Signature | Customer draws signature on canvas | Signature captured and preview shown |  |
| User converts lead to quote, customer approves via portal, then becomes WO | TC-3.10 Customer Pays Deposit | Customer pays deposit via card | Deposit invoice auto-created, payment recorded, confirmation shown |  |
| User converts lead to quote, customer approves via portal, then becomes WO | TC-3.11 Quote Locked | Admin opens same quote | Status=approved, locked_at timestamp present, lines finalized (4 total incl. ticked optional), deposit payment in history |  |
| User converts lead to quote, customer approves via portal, then becomes WO | TC-3.12 Create WO from Quote | Click "Create work order" on approved quote | WO created with all 4 lines copied, created_from=Q_xxxxx, bidirectional links (Quote ↔ WO) |  |
| User converts lead to quote, customer approves via portal, then becomes WO | TC-3.13 Schedule Visit | Assign tech + set date/time on WO | Visit created, appears on dispatch board |  |
| User converts lead to quote, customer approves via portal, then becomes WO | TC-3.14 Verify Bidirectional Links | Check WO → Related → Quote, Quote → Related → WO, Lead → Converted → Quote | Full chain Lead → Quote → WO all linked |  |
| User converts lead to quote, customer approves via portal, then becomes WO | TC-3.15 Tech Sees Job | Login as tech in PWA | New job from quote → WO visible in My Day with customer info, lines, schedule |  |

**Reference:** [StrongHeron QA Test Cases.md](../../StrongHeron%20QA%20Test%20Cases.md) — Phase 3

---

## Step 3.01 — New Lead Intake

**Test Scenario:** Step 3.01 — New Lead Intake

**App:** Admin → Leads (`/leads`)

| Test Step | Result |
|--------|----------|
| New lead exists or create manually (NewLeadModal) | Lead with status = `new` |

**NewLeadModal fields:** (same as Phase 2 Step 2.01)

| Field | Type | Example Data |
|-------|------|-------------|
| Name | input text | "David Chen" |
| Phone | input text | "+15125551234" |
| Email | input email | "davixample.comd@e" |
| Source | select | "Web" |
| Urgency | select | "normal" |
| Summary | textarea | "Need new AC unit installed, want quote first" |

---

## Step 3.02 — Convert Lead → Quote

**Test Scenario:** Step 3.02 — Convert Lead → Quote

**App:** Admin → Lead Record

| Test Step | Result |
|--------|----------|
| Click "Create quote" on lead record | `convertLead(id, { createQuote: true })` called |
| Quote builder opens | Customer prefilled from lead data |
| Lead status = `converted` | Link to quote shown on lead record |
| `convertedQuoteKey` present | Bidirectional link |

---

## Step 3.03 — Build Quote — Add Line Items

**Test Scenario:** Step 3.03 — Build Quote — Add Line Items

**App:** Admin → Quote New / Quote Record (`/quotes/new` or `/quotes/:id`)

**Quote form fields:**

| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| Customer | select | ✅ | Prefilled from lead conversion |
| Title | input text | - | "AC Installation Quote Summer 2025" |
| Expiration date | date | - | 30 days from today |
| Notes | textarea | - | "Internal notes — customer prefers morning install" |

**Line items (repeatable row):**

| Field | Type | Example Data |
|-------|------|-------------|
| Item (select from catalog) | select | "AC Unit" |
| Description | input text | Override description if needed |
| Qty | number | "1" |
| Rate | number (dollars) | "$2,500.00" (from catalog) |
| Optional | checkbox | ☐ unchecked |

| Test Step | Result |
|--------|----------|
| Add line "AC Unit" qty=1, rate=$2500 | Subtotal = $2,500.00 |
| Add line "Installation Labor" qty=1, rate=$450 (Preferred price) | Subtotal = $2,950.00 |
| Add line "Trip Fee" qty=1, rate=$75 | Subtotal = $3,025.00 |

**Totals section:**

| Label | Value |
|-------|-------|
| Subtotal | $3,025.00 |
| Tax | $0.00 (depends on taxable config) |
| Total | $3,025.00 |

---

## Step 3.04 — Add Optional Add-on

**Test Scenario:** Step 3.04 — Add Optional Add-on

**App:** Admin → Quote Record → Lines

| Test Step | Result |
|--------|----------|
| Add line "Loyalty Discount" rate=-$50 | Line added |
| Check "Optional" checkbox on Loyalty Discount line | Line marked as optional |
| Total does NOT change | Optional line excluded from initial total |
| Total remains $3,025.00 | Customer can tick/untick in portal |

---

## Step 3.05 — Set Deposit Required

**Test Scenario:** Step 3.05 — Set Deposit Required

**App:** Admin → Quote Record → Deposit section

**Deposit fields:**

| Field | Type | Example Data |
|-------|------|-------------|
| Required (checkbox) | checkbox | ✅ checked |
| Deposit % | number (1-100) | "50" |

| Test Step | Result |
|--------|----------|
| Enable deposit required | Deposit section active |
| Set 50% | Deposit amount = $1,512.50 (50% of $3,025) |

**Updated totals section:**

| Label | Value |
|-------|-------|
| Subtotal | $3,025.00 |
| Tax | $0.00 |
| Total | $3,025.00 |
| Deposit (50%) | $1,512.50 |

---

## Step 3.06 — Send Quote to Customer

**Test Scenario:** Step 3.06 — Send Quote to Customer

**App:** Admin → Quote Record

| Test Step | Result |
|--------|----------|
| Click "Send" in header | Quote status = `sent` |
| Portal link active | `/p/:token` URL available |
| Customer receives notification (email/SMS) | Link to portal sent |

---

## Step 3.07 — Customer Opens Portal

**Test Scenario:** Step 3.07 — Customer Opens Portal

**App:** Quote Portal (`/p/:token`)

| Test Step | Result |
|--------|----------|
| Customer opens portal link | Approval page displayed |
| Line items visible | AC Unit, Installation Labor, Trip Fee |
| Optional add-on visible with checkbox | Loyalty Discount can be ticked |
| Total displayed | $3,025.00 |
| Deposit amount displayed | $1,512.50 |

---

## Step 3.08 — Customer Ticks Optional Add-on

**Test Scenario:** Step 3.08 — Customer Ticks Optional Add-on

**App:** Quote Portal

| Test Step | Result |
|--------|----------|
| Customer checks "Loyalty Discount" (-$50) | Total recalculates server-side |
| Total changes | $3,025.00 - $50.00 = $2,975.00 |
| Deposit recalculates | $1,487.50 (50% of $2,975) |

---

## Step 3.09 — Customer Signature

**Test Scenario:** Step 3.09 — Customer Signature

**App:** Quote Portal → Signature section

| Test Step | Result |
|--------|----------|
| Customer draws signature on canvas/pad | Signature captured |
| Signature preview shown | Visual confirmation |

---

## Step 3.10 — Customer Pays Deposit

**Test Scenario:** Step 3.10 — Customer Pays Deposit

**App:** Quote Portal → Payment section

| Test Step | Result |
|--------|----------|
| Customer pays deposit via card | Payment processed |
| Deposit Invoice auto-created | Invoice for deposit amount |
| Payment record created | Payment linked to deposit invoice |
| Confirmation shown | "Thank you, deposit received" |

---

## Step 3.11 — Quote Locked (Admin Verify)

**Test Scenario:** Step 3.11 — Quote Locked (Admin Verify)

**App:** Admin → Quote Record

| Test Step | Result |
|--------|----------|
| Open same quote | Status = `approved` |
| `locked_at` timestamp present | Quote cannot be edited |
| Lines finalized (including optional ticked by customer) | 4 lines total |
| Deposit payment recorded | Payment history shown |

---

## Step 3.12 — Create WO from Approved Quote

**Test Scenario:** Step 3.12 — Create WO from Approved Quote

**App:** Admin → Quote Record

| Test Step | Result |
|--------|----------|
| Click "Create work order" | WO created |
| WO lines = lines from quote | All 4 lines copied |
| `created_from` = `Q_xxxxx` | Source quote recorded |
| Quote record → Related tab → link to WO | Bidirectional |
| WO record → Related tab → link to Quote | Bidirectional |

---

## Step 3.13 — Schedule Visit on WO

**Test Scenario:** Step 3.13 — Schedule Visit on WO

**App:** Admin → WO Record

**Schedule fields:**

| Field | Type | Example Data |
|-------|------|-------------|
| Date | date picker | Tomorrow |
| Start time | time picker | "08:00" |
| End time | time picker | "12:00" |
| Assigned tech | select | Select tech from Phase 1 |

| Test Step | Result |
|--------|----------|
| Assign tech + set schedule → Save | Visit created |
| Visit appears on Dispatch board | Card shown in tech's row |

---

## Step 3.14 — Check Related Links (Bidirectional)

**Test Scenario:** Step 3.14 — Check Related Links (Bidirectional)

**App:** Admin

| Test Step | Result |
|--------|----------|
| WO → Related tab | Link to Quote visible (`Q_` key) |
| Quote → Related tab | Link to WO visible (`WO_` key) |
| Lead → Converted section | Link to Quote visible |
| Full chain: Lead → Quote → WO | All linked |

---

## Step 3.15 — Tech Views New Job in PWA

**Test Scenario:** Step 3.15 — Tech Views New Job in PWA

**App:** PWA → `/today`

| Test Step | Result |
|--------|----------|
| Login as tech | My Day displayed |
| New job (from quote → WO) appears | Visit card with customer info, location, time |
| Tap → Job detail | Customer info, lines, schedule visible |

---

## ✅ Phase 3 Results

| Item | Status |
|------|--------|
| Lead intake |  |
| Convert lead → Quote (not WO) |  |
| Build quote with line items |  |
| Optional add-on line |  |
| Deposit required 50% |  |
| Send quote → customer portal |  |
| Customer reviews, ticks add-on, signs |  |
| Customer pays deposit |  |
| Quote locked (approved) |  |
| Create WO from approved quote |  |
| Related links bidirectional (Lead ↔ Quote ↔ WO) |  |
| Tech views job in PWA |  |

**Proceed to Phase 4: Tech executes the job in the field.**
