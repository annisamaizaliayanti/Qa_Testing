# Phase 2 — QA Test Results

## Step 5.01 — Open Checkout Screen

**Test Scenario:** Step 5.01 — Open Checkout Screen

**App:** PWA → `/job/:id/checkout`

| Test Step | Result |
|--------|----------|
| Tap "Checkout" from job screen | Pass — Checkout screen opened after completing the visit |
| AppHeader: "Checkout" + WO key | Pass — Header displays Checkout and WO-0100 |
| Subtotal | $275.00	Pass |
| Tax | Not displayed	Pass — No tax amount is shown |
| Total | $275.00	Pass |
| Deposit paid | Not displayed	Pass — No deposit is shown |
| Balance due | $275.00	Pass — Balance due is highlighted |

---

## Step 5.02 — Add Field Item (Optional)

**Test Scenario:** Step 5.02 — Add Field Item (Optional)

**App:** PWA → Checkout Screen → Materials section

| Test Step | Result |
|--------|----------|
| Tap "Add item" | Pass — Add item form opened with Description, Quantity, Rate, and Category fields |
| Add material used (e.g. extra part) | Pass — Extra AC Filter was added to the work order. Two entries were created: $20.00 and $25.00. | 
| Total recalculates | Pass — Total increased from $275.00 to $320.00. Balance due recalculation is not yet verified |

---

## Step 5.03 — Sign-off Checkout Document

**Test Scenario:** Step 5.03 — Sign-off Checkout Document

**App:** PWA → Sign-offs (from checkout flow)

| Test Step | Result |
|--------|----------|
| View pending sign-offs for stage checkout | Pass — Change Authorization appeared with Awaiting status and a $45.00 NTE overage |
| Tap document → review content | Pass — Review screen displayed work order details, line items, totals, and photos |
| Draw tech signature | Pass — Technician signature was captured successfully |
| Submit | Pass — Change Authorization status changed to Signed |

---

## Step 5.04 — Customer Review Screen Before Signing

**Test Scenario:** Step 5.04 — Customer Review Screen Before Signing

| Test Step | Result |
|--------|----------|
| Navigate to sign-off review | Pass — Review screen opened before signature capture |
| Work summary displayed | Pass — Work order title, customer name, and work summary were displaye |
| Line items displayed | Pass — All five work order line items with descriptions, quantities, and amounts were displayed |
| Totals displayed | Pass — Subtotal ($300.00) and Total ($300.00) were displayed. No tax or deposit was applied |
| Photos displayed | Pass — Before and After photos were displayed |
| Scroll down → "Sign" button | Pass — "Proceed to Sign" button was available |
| Tap "Sign" | Pass — Navigated to the signature capture screen |

---

## Step 5.05 — Customer Signature (On-Device)

**Test Scenario:** Step 5.05 — Customer Signature (On-Device)

| Test Step | Result |
|--------|----------|
| Customer signs on tech's device | Pass — Signature canvas opened on the technician's device |
| Signer name = customer name | Pass — Signer name was entered manually |
| Capture signature → Submit | Pass — Signature was drawn on the canvas and submitted successfully |
| Signed document stores: signerName, signedAt, method: 'on_device' | Not Tested — Signature metadata was not verified |

---

## Step 5.06 — Remote Signing Flow

**Test Scenario:** Step 5.06 — Remote Signing Flow

| Test Step | Result |
|--------|----------|
| Tech taps "Send for remote signing" | Pass — Remote signing form and Send button were available |
| Customer receives SMS/email with signing link | Pass — Customer received an email containing the Review & Sign link for WO-0102 |
| WO status shows "Awaiting signature" | Observation — Sign-off document displayed Awaiting signature before signing |
| Tech can leave site | Not Tested — Visit behavior while awaiting remote signature was not verified |
| Customer opens link → same review screen (summary, lines, totals, photos) | Fail — Review & Sign link displayed "Link unavailable" with an unexpected HTML response instead of JSON |
| Customer types name + signs on their phone | Not Tested — Customer could not access the remote signing page |
| Signed document stores: method: 'remote' | Not Tested — Remote signing metadata was not verified |
| Office can "Resend" if customer hasn't signed | Pass — Resend button was available |
| After customer signs → WO status updates from "Awaiting signature" | Not Tested — Status update after remote signing was not verified |

---

## Step 5.07 — Signed PDF Immutable + Metadata

**Test Scenario:** Step 5.07 — Signed PDF Immutable + Metadata

| Test Step | Result |
|--------|----------|
| Open signed document from WO | Pass — Customer Sign-off PDF opened successfully from WO-0100 |
| Content = snapshot at signing time (line items, totals) | Observation — PDF displayed work order details and customer confirmation statement; line items and totals were not displayed |
| Metadata stored: signer name | Pass — Signer name "David che" was displayed |
| Metadata stored: timestamp | Pass — Signing timestamp was displayed as November 2, 2026 at 04:47 PM |
| Metadata stored: method | Pass — Signing method was displayed as On-device |
| Try editing WO lines after signing → open PDF again | Not Tested — PDF does not display line items or totals, so changes to work order amounts cannot be compared |

---

## Step 5.08 — Collect Payment

**Test Scenario:** Step 5.08 — Collect Payment

| Test Step | Result |
|--------|----------|
| Select method (e.g. "cash") | Pass — Cash payment method was selected |
| Fill reference (optional) | Pass — Reference field was left blank as it is optional |
| Tap "Record $X,XXX.XX" | Pass — Record $440.00 was submitted |
| Payment recorded | Pass — "Cash/check recorded" confirmation was displayed |

---

## Step 5.09 — Done Screen

**Test Scenario:** Step 5.09 — Done Screen

**App:** PWA → `/job/:id/done`

| Test Step | Result |
|--------|----------|
| After payment → tap "Complete job" | Fail — The app displayed "Could not complete job — please try again" |
| Auto-navigate to Done screen | Not Tested — The app did not navigate to the Done screen |
| Summary displayed | Not Tested — Completion summary was not displayed |
| State passed: { payment, workOrderKey } | Not Tested — State data was not verified |

---

## Step 5.10 — Verify in Admin: WO Complete

**Test Scenario:** Step 5.10 — Verify in Admin: WO Complete

**App:** Admin → Work Order Record

| Test Step | Result |
|--------|----------|
| Open same WO in admin | Pass — WO-0100 record was displayed in Admin |
| Status = complete | Fail — Status displayed as invoiced, not complete |
| completed_at timestamp present | Not Tested — Completion timestamp was not verified |
| Visit status = completed | Not Tested — Visit status was not independently verified |

---

| Test Step | Result |
|--------|----------|
| WO record → view payment section | Pass — Payment of $440.00 was recorded for David Che |
| Payment amount correct | Pass — Payment amount matches the $440.00 balance due |
| Payment method correct | Pass — Payment method displayed as cash |
| Reference/note shown | Pass — Reference and note were left blank; both were optional |
| Payments page → filter by customer | Pass — Payment GN01BHQF appeared for David Che |

---

## Step 5.12 — WO → Create Invoice

**Test Scenario:** Step 5.12 — WO → Create Invoice

**App:** Admin → WO Record

| Test Step | Result |
|--------|----------|
| Click "Create invoice" on WO record | Fail — Invoice creation was blocked with the message: "Work order must be complete before creating an invoice" |
| Invoice lines = WO lines | Not Tested — Invoice was not created |
| created_from = WO key | Not Tested — Invoice source linkage could not be verified |
| Invoice status = draft | Not Tested — No invoice draft was created |
| Invoice amount = remaining balance (or full if no deposit) | Not Tested — Invoice amount could not be verified |

---

## Step 5.13 — Finalize + Send Invoice

**Test Scenario:** Step 5.13 — Finalize + Send Invoice

**App:** Admin → Invoice Record (`/invoices/:id`)

| Test Step | Result |
|--------|----------|
| Review invoice draft | Not Tested — Invoice draft was not created |
| Click "Send" | Not Tested — Invoice could not be sent because no draft was available | 
| Invoice locked | Not Tested — Invoice locking could not be verified |
| Customer can pay in portal | Not Tested — Portal payment link was not generated or verified |
| Customer gets notification | Not Tested — Invoice notification was not sent or verified |

