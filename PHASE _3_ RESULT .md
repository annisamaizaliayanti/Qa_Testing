# Phase 3 — QA Test Results


## Step 3.01 — New Lead Intake

**Test Scenario:** Step 3.01 — New Lead Intake

**App:** Admin → Leads (`/leads`)

| Test Step | Result |
|--------|----------|
| New lead exists or create manually (NewLeadModal) | Pass — Lead David Chen successfully created with status new |

---

## Step 3.02 — Convert Lead → Quote

**Test Scenario:** Step 3.02 — Convert Lead → Quote

**App:** Admin → Lead Record

| Test Step | Result |
|--------|----------|
| Click "Create quote" on lead record | Pass — Quote QUO-0026 berhasil dibuat dari lead David Che |
| Quote builder opens | Pass — Quote builder terbuka dengan nama David Che |
| Lead status = converted | Pass — As confirmed, the lead conversion process completed successfully |
| convertedQuoteKey present | Pass — Quote QUO-0026 was successfully linked to the lead |
| Customer approves quote | Pass — Quote status changed to approved and the approval confirmation was displayed |

---

## Step 3.03 — Build Quote — Add Line Items

**Test Scenario:** Step 3.03 — Build Quote — Add Line Items

**App:** Admin → Quote New / Quote Record (`/quotes/new` or `/quotes/:id`)

| Test Step | Result |
|--------|----------|
| Add line "AC Unit" qty=1, rate=$2500 | Observation — Actual displayed rate was $250.00, while expected rate was $2,500.00 |
| Add line "Installation Labor" qty=1, rate=$450 (Preferred price) | Observation — Actual displayed rate was $250.00, while expected rate was $450.00 |
| Add line "Trip Fee" qty=1, rate=$75 | Pass — Rate of $75.00 was accepted |

---

## Step 3.04 — Add Optional Add-on

**Test Scenario:** Step 3.04 — Add Optional Add-on

**App:** Admin → Quote Record → Lines

| Test Step | Result |
|--------|----------|
| Add line "Loyalty Discount" rate=-$50 | Fail — Loyalty Discount is not available in the item selection list |
| Check "Optional" checkbox on Loyalty Discount line | Not Tested — The item is unavailable |
| Total does NOT change | Not Tested — Optional item could not be added |
| Total remains $3,025.00 | Not Tested — Optional discount functionality could not be verified |

---

## Step 3.05 — Set Deposit Required

**Test Scenario:** Step 3.05 — Set Deposit Required

**App:** Admin → Quote Record → Deposit section

| Test Step | Result |
|--------|----------|
| Enable deposit required | Fail — No deposit configuration or Required checkbox is available on the Quote Record page |
| Set 50% | Fail — No Deposit Percentage field is available to enter the required 50% |

---

## Step 3.06 — Send Quote to Customer

**Test Scenario:** Step 3.06 — Send Quote to Customer

**App:** Admin → Quote Record

| Test Step | Result |
|--------|----------|
| Click "Send" in header | Pass — Quote was sent to the customer for review and approval |
| Portal link active (/p/:token) | Pass — Portal link was available and opened successfully |
| Customer receives notification (email/SMS) | Pass — Customer received the quote notification and could open the portal link to review and approve the quote |

---

## Step 3.07 — Customer Opens Portal

**Test Scenario:** Step 3.07 — Customer Opens Portal

**App:** Quote Portal (`/p/:token`)

| Test Step | Result |
|--------|----------|
| Customer opens portal link | Pass — David's quote portal opened successfully |
| Line items visible | Fail — Displayed AC Tune-Up TC1, Site Labor, and AC Tune-Up TC2 instead of AC Unit, Installation Labor, and Trip Fee |
| Optional add-on visible with checkbox | Fail — Loyalty Discount checkbox was not displayed |
| Total displayed | Fail — Actual total was $275.00 instead of $3,025.00 |
| Deposit amount displayed | Fail — Actual deposit was $0.00 instead of $1,512.50 | 

---

## Step 3.08 — Customer Ticks Optional Add-on

**Test Scenario:** Step 3.08 — Customer Ticks Optional Add-on

**App:** Quote Portal

| Test Step | Result |
|--------|----------|
| Customer checks "Loyalty Discount" (-$50) | Blocked — Loyalty Discount checkbox is not available on the quote portal |
| Total changes | Not Tested — The optional discount could not be selected |
| Deposit recalculates | Not Tested — Deposit recalculation could not be verified |

---

## Step 3.09 — Customer Signature

**Test Scenario:** Step 3.09 — Customer Signature

**App:** Quote Portal → Signature section

| Test Step | Result |
|--------|----------|
| Customer draws signature on canvas/pad | Pass — Customer signature was successfully captured |
| Signature preview shown | Pass — Signature preview was displayed as visual confirmation |

---

## Step 3.10 — Customer Pays Deposit

**Test Scenario:** Step 3.10 — Customer Pays Deposit

**App:** Quote Portal → Payment section

| Test Step | Result |
|--------|----------|
| Customer pays deposit via card | Fail — No card payment option is available on the quote portal |
| Deposit Invoice auto-created | Not Tested — Deposit payment could not be initiated |
| Payment record created | Not Tested — No deposit payment was completed |
| Confirmation shown | Fail — The message "Thank you, deposit received" was not displayed |

---

## Step 3.11 — Quote Locked (Admin Verify)

**Test Scenario:** Step 3.11 — Quote Locked (Admin Verify)

**App:** Admin → Quote Record

| Test Step | Result |
|--------|----------|
| Open same quote | Pass — Quote status is approved |
| locked_at timestamp present | Pass — Lock timestamp is present |
|  Lines finalized (including optional ticked by customer) | Fail — Finalized quote does not include the expected AC Unit, Installation Labor, Trip Fee, and optional Loyalty Discount | 
| Deposit payment recorded | Not Tested — No deposit payment was completed |

---

## Step 3.12 — Create WO from Approved Quote

**Test Scenario:** Step 3.12 — Create WO from Approved Quote

**App:** Admin → Quote Record

| Test Step | Result |
|--------|----------|
| Click "Create work order" | Pass — Work Order WO-0100 was successfully created for David Che |
| WO lines = lines from quote | Pass — Work Order contains the line items from the source quote |
| created_from = Q_xxxxx | Pass — WO-0100 is linked to source Quote QUO-0032 |
| Quote record → Related tab → link to WO | Pass — Quote QUO-0032 displays the related Work Order |
| WO record → Related tab → link to Quote | Pass — WO-0100 displays Quote QUO-0032 under Related |

---

## Step 3.13 — Schedule Visit on WO

**Test Scenario:** Step 3.13 — Schedule Visit on WO

**App:** Admin → WO Record

| Test Step | Result |
|--------|----------|
| Assign tech + set schedule → Save | Pass — Visit created for WO-0100, assigned to izza, on November 3, 2026, from 08:00 to 12:00 |
| Visit appears on Dispatch board | Pass — WO-0100 appeared on the November 3 Dispatch Board and opened its schedule details |

---

## Step 3.14 — Check Related Links (Bidirectional)

**Test Scenario:** Step 3.14 — Check Related Links (Bidirectional)

**App:** Admin

| Test Step | Result |
|--------|----------|
| WO → Related tab | Pass — WO-0100 displays the linked Quote QUO-0032 |
| Quote → Related tab | Pass — Quote QUO-0032 displays the linked Work Order WO-0100 |
| Lead → Converted section | Pass — Lead David Che displays the converted Quote link |
| Full chain: Lead → Quote → WO | Pass — Lead David Che, Quote QUO-0032, and WO-0100 are linked |

---


