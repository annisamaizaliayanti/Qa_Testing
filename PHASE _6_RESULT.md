# Phase 6 — QA Test Results


## Step 6.00 — Prepare: Create Items in Catalog

| Test Step | Result |
|--------|----------|
| Create 3 items: AC Unit - Full Replacement ($2,500), Labor - Installation ($400), Refrigerant R-410A ($300) | Pass — All 3 items were successfully saved in the catalog with the specified prices |

---

| Test Step | Result |
|--------|----------|
| Scroll to DEPOSIT section in right panel | Pass — The DEPOSIT section was visible in the WO record |
| Click No in Required row | Pass — The No/Yes toggle appeared | 
| Click Yes | Pass — Required changed to Yes |
| Amount row appears with "— Set amount" | Pass — The Amount row appeared |
| Click — Set amount | Pass — The amount input field opened with a $ prefix |
| Type 500 → press Enter or click ✓ | Pass — $500 was entered. Save confirmation is pending until ✓ is clicked |
| WO Record after reload | Pass — Required: Yes, Amount: $500.00, Invoice: —, Paid: Unpaid |
| Create & send deposit invoice button appears | Pass — Button was displayed |

---

## Step 6.02 — Schedule Visit + Set Status In Progress

| Test Step | Result |
|--------|----------|
| Click Visits tab | Pass — Visits tab displayed |
| Click + Schedule a visit | 
| Pass — Schedule form appeared | 
| Fill date + assigned tech → Save | Pass — Visit created for izza on 2026-11-04, status Scheduled |
| PATCH status → in_progress | Blocked — Backend/API access was not provided; status update could not be performed | 

---

## Step 6.03 — Tech Views Job — Deposit Blocked

**Test Scenario:** Step 6.03 — Tech Views Job — Deposit Blocked

**App:** PWA → Login as tech → `/today` (My Day)

| Test Step | Result |
|--------|----------|
| Tech opens PWA, logs in | Pass — Tech dashboard displayed |
| WO appears in My Day | Pass — Scheduled job for isa appeared in My Day |
| Tap job → open /job/:workOrderId | Pass — Job screen displayed |
| Deposit gate active | Pass — Banner displayed: “Deposit $500.00 required — not paid. Contact office” |
| wo.depositRequired && !wo.depositPaid → true | Not Tested — Source code (JobScreen.tsx) was not provided, so the internal condition could not be verified |

---

## Step 6.04 — Tech Cannot Start

**Test Scenario:** Step 6.04 — Tech Cannot Start

**App:** PWA → Job Screen

| Test Step | Result |
|--------|----------| 
| "Start visit" button NOT shown | Pass — Start visit button was not displayed; deposit gate warning appeared instead |
| Tech cannot start work | Pass — Technician could not start the visit because the Start visit button was unavailable |
| Tap "Message office" (if available) | Pass — Messages screen opened and displayed quick-message options, including “Waiting on deposit confirmation”|

---

## Step 6.05 — Admin Creates & Sends Deposit Invo## Step 6.01 — Create WO with Line Items

| Test Step | Result |
|--------|----------|
| Open WO Record → right panel DEPOSIT section | Pass — Deposit amount of $500.00 was displayed |
| Click Create & send deposit invoice | Pass — Invoice confirmation was completed |
| Confirm → Submit | Pass — Deposit invoice INV-0023 was created and sent |
| Invoice key shown on WO Record | Pass — Invoice number INV-0023, amount $500.00 |

---

## Step 6.06 — Admin Processes Deposit Payment

| Test Step | Result |
|--------|----------|
| Payment recorded to deposit invoice | Pass — $500.00 payment was recorded successfully |
| WO Record → DEPOSIT section | Pass — Deposit status changed to ✓ Paid |
| wo.depositPaid = true | Not Tested — Internal code could not be verified; deposit status changed to Paid in the UI |

---

## Step 6.07 — Tech Can Now Start

| Test Step | Result |
|--------|----------|
| Tech refreshes / re-opens job | Pass — Deposit gate disappeared; deposit status displayed as Paid |
| wo.depositPaid = true | Not Tested — Internal code condition could not be verified directly |
| "Start visit" button appears | Pass — Start visit button appeared |
| Tap Start visit | Observation — Documents screen opened instead of navigating to Time |
| Timer starts running | Pass — Timer started successfully after clicking Start timer in the Time menu |

---

## Step 6.08 — Pre-work Sign-off Required (NTE Capture) 

| Test Step | Result |
|--------|----------|
| Doc template created | Blocked — Backend/API access unavailable |
| Navigate to /job/:id/signoff in PWA | Pass — Sign-off screen displayed |
| Pre-work authorization document shown | Observation — Work Authorization Form displayed with work summary and total $2,900.00, but explicit NTE wording was not shown |
| Tech + customer sign | Pass — Signature was completed |
| Submit signatures | Pass — Work Authorization Form successfully signed |
| authorizedCents set = totalCents | Not Tested — Requires backend/source verification |

---

## Step 6.09 — Tech Works Job

**Test Scenario:** Step 6.09 — Tech Works Job

| Test Step | Result |
|--------|----------|
| Fill checklist items | Pass — All required checklist items were completed |
| Record time (timer or manual) | Pass — Time entry was successfully recorded in the PWA |
| Take photos (before/after) | Fail — The same photo uploaded as Before automatically appears in After photos. The photo also cannot be removed from the After section |
| Tile counts update | Pass — Progress counts were updated on the job screen |

---

## Step 6.10 — Total Exceeds NTE → Warning Appears

| Test Step | Result |
|--------|----------|
| Open Materials tab | Pass — Materials/line items list displayed |
| Tap + Add material | Pass — Add item form displayed |
| Test Step	Result
Open Materials tab	Pass — Materials/line items list displayed.
Tap + Add material	Pass — Add item form displayed.
Select "Refrigerant R-410A", qty 1, $300.00	 | Pass — Refrigerant R-410A was successfully added to WO-0103 |
| Total rises to $3,200 | Pass — Total increased from $2,900.00 to $3,200.00 | 
| Job screen HeroCard updates | Pass — Warning displayed: "Total exceeds NTE by $300.00". Customer Change Authorization is required before checkout |
|  wo.authorizedCents ($290,000) < wo.totalCents ($320,000) | Not Tested — Internal values and overNte condition cannot be directly verified through the UI |

---

## Step 6.11 — Checkout Blocked Due to Over-NTE

| Test Step | Result |
|--------|----------|
| Tech tries to open checkout | Fail — Checkout screen opened despite the NTE overage |
| "Costs exceed authorized amount" banner | Fail — NTE warning appears on Materials and Job screens, but not on Checkout |
| Checkout cannot proceed | Fail — Collect $2,200.00 button remains available |
| wo.authorizedCents > 0 && wo.totalCents > wo.authorizedCents | Not Tested — Internal condition cannot be verified through the UI |

---

## Step 6.12 — Change Authorization (Delta Auth)


| Test Step | Result |
|--------|----------|
| Navigate to Sign-offs in PWA | Pass — Change Authorization document displayed with $300.00 · over NTE |
| Body shows delta amount (+$300.00) | Observation — Document list shows +$300.00, but Review body does not explicitly state the additional amount |
| Customer signs | Pass — Change Authorization signature submitted successfully |
| Submit → signed document created | Pass — Document signed successfully |
| authorizedCents ≥ totalCents | Pass (UI evidence) — NTE increased to $3,200.00, matching the work order total. Internal variable itself was not directly inspected |

---

## Step 6.13 — Admin Override NTE Gate (Alternative)

| Test Step | Result |
|--------|----------|
| authorizedCents = totalCents | Blocked — API override requires backend access | 
| WO Record shows authorized = $3,200.00 | Not Tested — Admin WO Record has not been checked for override details |
| Tech can proceed to checkout | Not Tested — Checkout after admin override has not been tested |

---

## Step 6.14 — Signed PDF Immutable After Pre-work Auth

| Test Step | Result |
|--------|----------|
| Open WO → view signed documents | Pass — Signed pre-work authorization displayed |
| Try editing WO field (summary, notes) | Pass — Summary and Notes were successfully edited |
| Open signed pre-work auth PDF | Pass — PDF retained its original content after WO data was edited |
| locked_at timestamp present in doc record |
| Not Tested — Internal metadata cannot be verified through the UI | 
| signerName, signedAt, method present in metadata | Pass — Signer name, signing date/time, and signing method are displayed |

---

## Step 6.15 — Checkout Now Succeeds

| Test Step | Result |
|--------|----------|
| NTE warning gone | Pass — NTE warning no longer appears after Change Authorization was signed |
| Normal checkout flow | Pass — Charges and totals are displayed ($3,200.00) |
| Sign-off checkout documents (if any) | Pass — Customer Sign-off document is signed and displayed in Sign-offs |
| Collect payment | Not Tested — Checkout payment has not been collected |
| Balance = $3,200 - $500 deposit = $2,700 | Fail — Application displays $2,200.00 instead of $2,700.00 | 
| Click Complete | Pass — Paid confirmation screen appeared |
| WO status → complete | Pass — WO-0103 status changed to Complete |
