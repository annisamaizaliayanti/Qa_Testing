# Phase 2 — Lead Intake → Convert to Customer + Work Order

> **Concept:** An incoming lead (from phone/email/web) is reviewed by admin, then converted directly into a new Customer + Work Order without going through a Quote.
> After the WO is created, assign a tech, and the tech can see the job in the PWA.

**Prerequisite:** Phase 1 complete (admin, tech, items, checklists already set up).

---

## QA Test Case Summary

| User Story | Test Scenario | Test Step | Result | Status |
|---|---|---|---|---|
| User converts a lead directly into a Work Order | TC-2.01 Lead Intake | Open Leads page; create lead manually via NewLeadModal (name, phone, email, source, urgency, summary) | Lead created with status=new, AI summary auto-generated |  |
| User converts a lead directly into a Work Order | TC-2.02 Review Lead Detail | Click lead from list | Lead record page shows source, contact info, summary, urgency, status badge "new" |  |
| User converts a lead directly into a Work Order | TC-2.03 Reply to Lead | View AI-drafted reply, approve/edit, send | Message sent to lead, status changes new → contacted, message logged in conversation history |  |
| User converts a lead directly into a Work Order | TC-2.04 Convert Lead → WO | Click "Create work order" on lead record | Customer auto-created (or matched by email/phone), WO created and linked to customer, lead status=converted, bidirectional links visible |  |
| User converts a lead directly into a Work Order | TC-2.05 Verify Customer | Navigate to customer from converted lead link | Customer record shows name, phone, email transferred from lead data, CUS_ key assigned |  |
| User converts a lead directly into a Work Order | TC-2.06 Verify Work Order | Navigate to WO from converted lead link | WO record shows customer name in header, created_from=lead, convertedWoKey visible on lead record |  |
| User converts a lead directly into a Work Order | TC-2.07 Assign Tech + Schedule | Assign tech, set date/time on WO schedule section | Visit created with status=scheduled |  |
| User converts a lead directly into a Work Order | TC-2.08 Dispatch Board | Open Dispatch board, filter today | Visit card appears in assigned tech's row with customer name, time, job type |  |
| User converts a lead directly into a Work Order | TC-2.09 Tech Sees Job in PWA | Login as tech in PWA, open /today | Visit card visible with customer, location, time; tap navigates to job detail |  |

**Reference:** [StrongHeron QA Test Cases.md](../../StrongHeron%20QA%20Test%20Cases.md) — Phase 2

---

## Step 2.01 — Lead Intake

**Test Scenario:** Step 2.01 — Lead Intake

**App:** Admin → Leads (`/leads`)

| Test Step | Result |
|--------|----------|
| Open Leads page | Lead list displayed |
| New lead exists (from email/missed call/manual) | Lead with status = `new` |
| AI summary auto-generated | Summary field populated |

**If no lead exists, create manually via NewLeadModal:**

| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| Name | input text | - | "Sarah Wilson" |
| Phone | input text | - | "+15125550000" |
| Email | input email | - | "sarah@example.com" |
| Source | select | - | From value-set lead sources (Web, Phone, Referral, etc.) |
| Urgency | select | - | low, normal, high, emergency |
| Summary | textarea | - | "AC not cooling, needs repair ASAP" |

| Test Step | Result |
|--------|----------|
| Fill form → Submit | Lead created with status = `new` |

---

## Step 2.02 — Review Lead Detail

**Test Scenario:** Step 2.02 — Review Lead Detail

**App:** Admin → Lead Record (`/leads/:id`)

| Test Step | Result |
|--------|----------|
| Click lead from list | Lead record page displayed |
| View all details | Source, contact info, summary, urgency visible |
| AI summary/draft available | AI-parsed information shown |
| Status badge = "new" | Status indicator clear |

---

## Step 2.03 — Reply to Lead

**Test Scenario:** Step 2.03 — Reply to Lead

**App:** Admin → Lead Record → Conversation section

| Test Step | Result |
|--------|----------|
| View AI-drafted reply | Suggested response shown |
| Approve / edit reply → Send | Message sent to lead |
| Status changes | new → contacted |
| Message recorded in conversation | History displayed |

**API calls:** `replyLead(id, message)`, `updateLead(id, { status: 'contacted' })`

---

## Step 2.04 — Convert Lead → Work Order Directly

**Test Scenario:** Step 2.04 — Convert Lead → Work Order Directly

**App:** Admin → Lead Record

| Test Step | Result |
|--------|----------|
| Click "Create work order" button | `convertLead(id, { createWo: true })` called |
| Customer auto-created (or matched by email/phone) | Customer record exists |
| Work Order created | WO linked to new customer |
| Lead status = `converted` | Badge changes |
| Converted section displayed | Link to WO (key) + Customer visible |

**Available convert options on LeadRecord:**
- **"Create work order"** → `convertLead(id, { createWo: true })` → Customer + WO
- **"Create quote"** → `convertLead(id, { createQuote: true })` → Customer + Quote (Phase 3)
- **"Convert to customer"** → `convertLead(id, { createCustomer: true })` → Customer only

---

## Step 2.05 — Verify Created Customer

**Test Scenario:** Step 2.05 — Verify Created Customer

**App:** Admin → Customer Record (navigate from converted lead link)

| Test Step | Result |
|--------|----------|
| Open customer from lead conversion | Customer record displayed |
| Lead info transferred | Name, phone, email from lead pre-filled |
| Customer has `CUS_` key | Key auto-generated |

---

## Step 2.06 — Verify Created Work Order

**Test Scenario:** Step 2.06 — Verify Created Work Order

**App:** Admin → Work Order Record (navigate from converted lead link)

| Test Step | Result |
|--------|----------|
| Open WO from converted lead | WO record page displayed |
| WO linked to new customer | Customer name visible in header |
| `created_from` = lead | Source metadata present |
| `convertedWoKey` shown on lead record | Bidirectional link |

---

## Step 2.07 — Assign Tech + Schedule Visit

**Test Scenario:** Step 2.07 — Assign Tech + Schedule Visit

**App:** Admin → Work Order Record → Schedule section

**Work Order form fields (if editing is needed):**

| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| Customer | select | ✅ | Select customer (already prefilled) |
| Site / location | select | - | Select site from customer |
| Assigned tech | select | - | Select technician |
| Job type | select | - | "AC Install" or "Maintenance" |
| Priority | select | - | From value-set priorities |
| Price list | select | - | Optional |
| Summary / description | textarea | - | "AC repair - not cooling" |
| Internal notes | textarea | - | "Customer prefers morning" |
| External ref | input text | - | "ERP-001" |
| Customer PO | input text | - | "PO-2024-001" |
| NTE amount | number | - | "5000.00" |

**Schedule fields:**

| Field | Type | Example Data |
|-------|------|-------------|
| Date | date picker | Today |
| Start time | time picker | "09:00" |
| End time | time picker | "11:00" |

**New site inline (if customer has no site yet):**

| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| Site label | input text | - | "Main office" |
| Street address | input text | ✅ | "123 Main St" |
| Line 2 | input text | - | "Unit, Floor…" |
| City | input text | ✅ | "Springfield" |
| State | input text (2 char) | ✅ | "CA" |
| ZIP | input text | ✅ | "90210" |

| Test Step | Result |
|--------|----------|
| Assign tech → Set date/time → Save | Visit created |
| Visit status = scheduled | Ready in dispatch |

---

## Step 2.08 — View on Dispatch Board

**Test Scenario:** Step 2.08 — View on Dispatch Board

**App:** Admin → Dispatch (`/dispatch`)

| Test Step | Result |
|--------|----------|
| Open Dispatch board | Scheduling board displayed — crews & visits by day |
| Filter today | Visit card appears |
| Visit in assigned tech's row | Card shows customer name, time, job type |

---

## Step 2.09 — Tech Views Job in My Day (PWA)

**Test Scenario:** Step 2.09 — Tech Views Job in My Day (PWA)

**App:** PWA → `/today`

| Test Step | Result |
|--------|----------|
| Login as tech | Redirect to My Day |
| Today's schedule displayed | Visit card appears with: customer, location, time |
| Tap visit card | Navigate to job detail |

---

## ✅ Phase 2 Results

| Item | Status |
|------|--------|
| Lead intake (manual or automatic) |  |
| Lead review + AI summary |  |
| Reply to lead |  |
| Convert lead → Customer + WO |  |
| Customer created from lead data |  |
| WO created + linked to customer |  |
| Tech assigned + visit scheduled |  |
| Dispatch board shows visit |  |
| Tech views job in PWA |  |

**Proceed to Phase 3: Lead → Quote → Customer Approval → WO.**
