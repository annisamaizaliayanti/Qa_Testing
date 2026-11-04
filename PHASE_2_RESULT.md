# Phase 2 — QA Test Results

## Step 2.01 — Lead Intake

**Test Scenario:** Step 2.01 — Lead Intake

**App:** Admin → Leads (`/leads`)

| Test Step | Result |
|--------|----------|
| Open Leads page | Pass - Leads page is accessible and the lead list is displayed |
| New lead exists (from email/missed call/manual) | Pass - New Lead feature is available and existing leads are displayed |
| AI summary auto-generated | Pass - Observation — AI draft feature is available, but automatic summary generation has not been verified |

**If no lead exists, create manually via NewLeadModal:**

| Test Step | Result |
|--------|----------|
| Open Leads page | Pass — Leads page displayed successfully |
| New lead exists (from email/missed call/manual) | Pass — New lead LEAD-0022 (Sarah Wilson) successfully created |
|  AI summary auto-generated | Pass — “AC not cooling, needs repair ASAP” automatically appears under What They Need |
| Fill form → Submit | Pass — Lead created with status new |

---

## Step 2.02 — Review Lead Detail

**Test Scenario:** Step 2.02 — Review Lead Detail

**App:** Admin → Lead Record (`/leads/:id`)

| Test Step | Result |
|--------|----------|
| Click lead from list | Pass — Lead record LEAD-0022 displayed successfully | 
| View all details | Pass — Contact information, request summary, source, and urgency are visible |
| AI summary/draft available | Observation — Source selected as Web Form during lead creation is displayed as Unknown in the Details page |
| Status badge = "new" | Pass — Status displays new |

---

## Step 2.03 — Reply to Lead

**Test Scenario:** Step 2.03 — Reply to Lead

**App:** Admin → Lead Record → Conversation section

| Test Step | Result |
|--------|----------|
| View AI-drafted reply | Pass — AI-generated SMS reply displayed |
| Approve / edit reply → Send | Pass — SMS successfully sent; status shows sent |
| Status changes (new → contacted) | Fail — Lead status remains new after the message was sent |
| Message recorded in conversation | Pass — Sent message is displayed in conversation history |

---

## Step 2.04 — Convert Lead → Work Order Directly

**Test Scenario:** Step 2.04 — Convert Lead → Work Order Directly

**App:** Admin → Lead Record

| Test Step | Result |
|--------|----------|
| Click "Create work order" button | Pass — Create Work Order form opened successfully |
| Customer auto-created (or matched by email/phone) | Pass — Customer record created and linked to the lead |
| Work Order created | Pass — Work Order successfully created and linked to the customer |
| Lead status = converted | Pass — Lead status changed to converted |
| Converted section displayed | Pass — Work Order and Customer links are displayed |

---

## Step 2.05 — Verify Created Customer

**Test Scenario:** Step 2.05 — Verify Created Customer

**App:** Admin → Customer Record (navigate from converted lead link)

| Test Step | Result |
|--------|----------|
| Open customer from lead conversion | Pass — Customer record Sarah Wilson successfully displayed |
| Lead info transferred | Pass — Name, phone number, and email successfully transferred from lead data |
| Customer has CUS_ key | Pass — Customer ID CUS-0034 automatically generated |

---

## Step 2.06 — Verify Created Work Order

**Test Scenario:** Step 2.06 — Verify Created Work Order

**App:** Admin → Work Order Record (navigate from converted lead link)

| Test Step | Result |
|--------|----------|
| Open WO from converted lead | Pass — Work Order WO-0096 berhasil dibuka |
| WO linked to new customer | Pass — Customer Sarah Wilson (CUS-0034) terhubung |
| created_from = lead | Pass — Work Order menampilkan from LEAD-0022 |
| convertedWoKey shown on lead record | Pass — WO-0096 tampil pada daftar Leads |

---

## Step 2.07 — Assign Tech + Schedule Visit

**Test Scenario:** Step 2.07 — Assign Tech + Schedule Visit

**App:** Admin → Work Order Record → Schedule section

**Work Order form fields (if editing is needed):**

 Test Step | Result |
|--------|----------|
| Assign tech → Set date/time → Save | Pass — Visit successfully created and assigned to technician izza on November 2, 2026, from 16:02 to 18:03 |
| Visit status = scheduled | Pass — Visit status displayed as Scheduled, and Work Order WO-0096 status changed to Scheduled (ready for dispatch) |
| New site inline (if customer has no site yet) | Pass — Site Main office successfully created and saved under Sarah Wilson’s customer record, with the address 23 Main St, Springfield, CA 90210 | 

---

## Step 2.08 — View on Dispatch Board

**Test Scenario:** Step 2.08 — View on Dispatch Board

**App:** Admin → Dispatch (`/dispatch`)

| Test Step | Result |
|--------|----------|
| Open Dispatch board | Pass — Dispatch Board displayed with scheduled work orders |
| Filter today | Pass — WO-0096 appeared on November 2 at 4:02 PM |
| Visit in assigned tech's row | Pass — WO-0096 displays Sarah Wilson, the job summary, and assigned technician izza |

---

## Step 2.09 — Tech Views Job in My Day (PWA)

**Test Scenario:** Step 2.09 — Tech Views Job in My Day (PWA)

**App:** PWA → `/today`

| Test Step | Result |
|--------|----------|
| Login as tech | Pass — Technician successfully logged in and accessed My Day |
| Today's schedule displayed | Pass — Visit card displays Sarah Wilson with the scheduled time of 4:02 PM |
| Tap visit card | Pass — Successfully navigated to WO-0096 job detail |