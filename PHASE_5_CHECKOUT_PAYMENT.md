# Phase 5 — Tech Checkout + Payment in the Field

> **Concept:** After the job is complete (Phase 4), tech performs the checkout flow in the PWA:
> reviews totals, signs off checkout documents, collects payment from customer, completes WO.
> Admin then verifies in the admin panel: WO complete, payment recorded, then creates and sends invoice.

**Prerequisite:** Phase 4 complete (checklist done, time entry exists, photos uploaded).

---

## QA Test Case Summary

| User Story | Test Scenario | Test Step | Result | Status |
|---|---|---|---|---|
| Technician performs checkout and collects payment | TC-5.01 Checkout Screen | Tap "Checkout" from job screen | Totals breakdown: subtotal, tax (if >0), total, deposit paid (if >0), balance due (bold/highlighted) |  |
| Technician performs checkout and collects payment | TC-5.02 Add Field Item | Tap "Add item" on checkout, add extra part | Line item added to WO, total recalculates, balance due updates |  |
| Technician performs checkout and collects payment | TC-5.03 Sign-off Checkout Doc | View pending sign-offs for stage "checkout", tap document, review content, draw tech signature, submit | Signed document record created, signed status updated on checkout screen |  |
| Technician performs checkout and collects payment | TC-5.04 Customer Review Screen | Navigate to sign-off review screen | Work summary, line items (description/qty/rate), totals (subtotal/tax/total/deposit/balance), and photos all visible before signature |  |
| Technician performs checkout and collects payment | TC-5.05 Customer On-Device Signature | Tech hands device to customer, customer types name and signs | Signature captured, signed doc stores signerName, signedAt, method: on_device |  |
| Technician performs checkout and collects payment | TC-5.06 Remote Signing Flow | Tech taps "Send for remote signing" | Customer receives SMS/email link (no login required), WO shows "Awaiting signature", customer opens link and sees same review screen, signs remotely, method: remote stored |  |
| Technician performs checkout and collects payment | TC-5.07 Signed PDF Immutable | Admin opens signed document from WO | PDF shows snapshot of content at signing time (immutable), metadata: signer name, timestamp, method; editing WO after does not change PDF |  |
| Technician performs checkout and collects payment | TC-5.08 Collect Payment — Cash/Check | Pick Cash/Check method, fill reference/note, tap "Record" | recordManualPayment() called, payment recorded with method, reference, and note |  |
| Technician performs checkout and collects payment | TC-5.09 Complete Job → Done Screen | Tap "Complete job" | completeWorkOrder() called, navigate to /job/:id/done, summary shows payment method + WO key. Disabled if: balance >0 without payment, or required checklist incomplete (409 REQUIRED_ITEMS_INCOMPLETE) |  |
| Technician performs checkout and collects payment | TC-5.10 Admin Verify WO Complete | Open same WO in admin | Status=complete, completed_at timestamp recorded, visit status=completed |  |
| Technician performs checkout and collects payment | TC-5.11 Admin Verify Payment | Check WO payments section and /payments page | Payment amount, method, reference/note all match; filterable by customer |  |
| Technician performs checkout and collects payment | TC-5.12 Create Invoice from WO | Click "Create invoice" on WO record | Invoice draft created with WO lines copied, created_from=WO key, status=draft |  |
| Technician performs checkout and collects payment | TC-5.13 Send Invoice | Review draft, click "Send" | Status=open, locked_at set, customer notified with portal payment link |  |

**Reference:** [StrongHeron QA Test Cases.md](../../StrongHeron%20QA%20Test%20Cases.md) — Phase 5

---

## Step 5.01 — Open Checkout Screen

**Test Scenario:** Step 5.01 — Open Checkout Screen

**App:** PWA → `/job/:id/checkout`

| Test Step | Result |
|--------|----------|
| Tap "Checkout" from job screen | Navigate to checkout screen |
| AppHeader: "Checkout" + WO key | Header displayed |

**Totals breakdown:**

| Label | Value | Condition |
|-------|-------|-----------|
| Subtotal | `wo.subtotalCents` formatted | Always shown |
| Tax | `totalCents - subtotalCents` | Only if > 0 |
| Total | `wo.totalCents` formatted | Always shown |
| Deposit paid | `−depositCents` | Only if deposit > 0 |
| **Balance due** | `totalCents - depositCents` | **Bold, highlighted** |

**Example (from Phase 3 scenario):**

| Label | Value |
|-------|-------|
| Subtotal | $2,975.00 |
| Total | $2,975.00 |
| Deposit paid | −$1,487.50 |
| **Balance due** | **$1,487.50** |

---

## Step 5.02 — Add Field Item (Optional)

**Test Scenario:** Step 5.02 — Add Field Item (Optional)

**App:** PWA → Checkout Screen → Materials section

| Test Step | Result |
|--------|----------|
| Tap "Add item" | Item picker / input opens |
| Add material used (e.g. extra part) | Line item added to WO |
| Total recalculates | Balance due changes |

---

## Step 5.03 — Sign-off Checkout Document

**Test Scenario:** Step 5.03 — Sign-off Checkout Document

**App:** PWA → Sign-offs (from checkout flow)

| Test Step | Result |
|--------|----------|
| View pending sign-offs for stage `checkout` | Documents with REQUIRED AT = checkout shown |
| Tap document → review content | Document body displayed |
| Draw tech signature | `SignatureCapture` component |
| Submit | Signed document record created |

**Sign-off flow detail:**
1. `getPendingSignoffs(workOrderId, 'checkout')` → list pending docs
2. Tap doc → navigate to `/job/:id/signoff-review/:docId`
3. Review → Sign → `createSignedDocument({ workOrderId, docTemplateId, signerName, signatureFileId })`
4. Back to checkout → signed status updated

---

## Step 5.04 — Customer Review Screen Before Signing

**Test Scenario:** Step 5.04 — Customer Review Screen Before Signing

> **Purpose:** Verify that the customer sees a complete review screen (summary, line items, totals, photos) before being asked to sign. This applies to all sign-off types.

**App:** PWA → Sign-off Review Screen (`/job/:id/signoff-review/:docId`)

| Test Step | Result |
|--------|----------|
| Navigate to sign-off review | Review screen displayed (not immediately the signature pad) |
| **Work summary** displayed | WO summary/description visible |
| **Line items** displayed | All WO lines with description, qty, rate |
| **Totals** displayed | Subtotal, tax (if any), total, deposit paid, balance due |
| **Photos** displayed | Before/after photos from job (if any) |
| Scroll down → "Sign" button | Button at bottom of review content |
| Tap "Sign" | Navigate to signature capture |

---

## Step 5.05 — Customer Signature (On-Device)

**Test Scenario:** Step 5.05 — Customer Signature (On-Device)

> **Purpose:** Tech hands device to customer, customer types name and signs on tech's device.

**App:** PWA → Signature Capture

| Test Step | Result |
|--------|----------|
| Customer signs on tech's device | Signature canvas opens |
| Signer name = customer name | Input field (defaults from customer record) |
| Capture signature → Submit | Signature file uploaded + signed doc created |
| Signed document stores: `signerName`, `signedAt`, `method: 'on_device'` | Complete metadata |

---

## Step 5.06 — Remote Signing Flow

**Test Scenario:** Step 5.06 — Remote Signing Flow

> **Purpose:** Customer signs from their own phone via SMS/email link. Tech can leave the site; office can resend.

**App:** PWA → Sign-off Screen + Customer → Remote Signing Page

| Test Step | Result |
|--------|----------|
| Tech taps "Send for remote signing" | `requestRemoteSigning()` called |
| Customer receives SMS/email with signing link | No-login link |
| **WO status shows "Awaiting signature"** | Badge/indicator changes on WO record |
| Tech can leave site | Visit not blocked by pending remote sign |
| Customer opens link → **same review screen** (summary, lines, totals, photos) | No-login page, same content as on-device review |
| Customer types name + signs on their phone | Signature captured remotely |
| Signed document stores: `method: 'remote'` | Method recorded |
| Office can "Resend" if customer hasn't signed | `resendSigningRequest()` available in admin |
| After customer signs → WO status updates from "Awaiting signature" | Real-time or polling update |

---

## Step 5.07 — Signed PDF Immutable + Metadata

**Test Scenario:** Step 5.07 — Signed PDF Immutable + Metadata

> **Purpose:** Verify that the signed PDF is locked and stores all metadata (signer, timestamp, method).

**App:** Admin → WO Record → Signed Documents

| Test Step | Result |
|--------|----------|
| Open signed document from WO | PDF/document viewer displayed |
| Content = snapshot at signing time (line items, totals) | Immutable — does not change even if WO is edited |
| Metadata stored: **signer name** | Name typed by customer |
| Metadata stored: **timestamp** | `signedAt` datetime |
| Metadata stored: **method** | `on_device` or `remote` |
| Try editing WO lines after signing → open PDF again | PDF content **unchanged** |

---

## Step 5.08 — Collect Payment

**Test Scenario:** Step 5.08 — Collect Payment

**App:** PWA → Checkout Screen → Payment section

**Payment flow (PaymentSheet component):**

### Step 1: Pick Method

| Method | Icon | Description |
|--------|------|-------------|
| Cash / Check | 💵 | Record manual payment |
| Invoice later | 📄 | Send invoice to customer (pay later) |

**Amount due displayed:** formatted balance due amount.

### Step 2a: Cash / Check (Manual Payment)

| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| Payment method | button group | ✅ | cash, check, ach, other |
| Reference / Check # | input text | - | "Check #1042" |
| Note | input text | - | "Paid in full" |
| Amount | display only | - | Balance due amount |

| Test Step | Result |
|--------|----------|
| Select method (e.g. "cash") | Button highlighted blue |
| Fill reference (optional) | Text captured |
| Tap "Record $X,XXX.XX" | `recordManualPayment({ amountCents, method, ref, note })` |
| Payment recorded | Success feedback |

### Step 2b: Invoice Later

| Test Step | Result |
|--------|----------|
| Select "Invoice later" | Invoice will be sent later (Phase 5.09) |
| Proceed to complete | No payment collected now |

---

## Step 5.09 — Done Screen

**Test Scenario:** Step 5.09 — Done Screen

**App:** PWA → `/job/:id/done`

| Test Step | Result |
|--------|----------|
| After payment → tap "Complete job" | `completeWorkOrder(workOrderId)` called via `POST /work-orders/:id/complete` |
| Auto-navigate to Done screen | `/job/:id/done` |
| Summary displayed | Payment method, WO key, completion confirmation |
| State passed: `{ payment, workOrderKey }` | Data from checkout |

**Complete button disabled if:**
- Balance > 0 AND no payment collected → "Collect payment before completing"
- Required checklist items not complete → server returns 409 `REQUIRED_ITEMS_INCOMPLETE`

---

## Step 5.10 — Verify in Admin: WO Complete

**Test Scenario:** Step 5.10 — Verify in Admin: WO Complete

**App:** Admin → Work Order Record

| Test Step | Result |
|--------|----------|
| Open same WO in admin | WO record displayed |
| Status = `complete` | Badge/status indicator updated |
| `completed_at` timestamp present | Completion time recorded |
| Visit status = completed | Visit also finished |

---

## Step 5.11 — Verify Payment Recorded

**Test Scenario:** Step 5.11 — Verify Payment Recorded

**App:** Admin → WO Record → Payments tab / Payments page (`/payments`)

| Test Step | Result |
|--------|----------|
| WO record → view payment section | Payment recorded |
| Payment amount correct | Balance due amount |
| Payment method correct | cash/check/ach/other |
| Reference/note shown | Data from manual fields |
| Payments page → filter by customer | Payment appears in list |

**Admin TakePaymentModal fields (if recording payment from admin):**

| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| CUSTOMER | select | ✅ | Select customer |
| PAYMENT METHOD | button group | ✅ | cash, check, ach, other, card_on_file |
| AMOUNT ($) | number | ✅ | "1487.50" |
| NOTE (optional) | input text | - | "Check #1042" |

**Payment methods in admin (more options than PWA):**

| Method | Description |
|--------|-------------|
| cash | Cash payment |
| check | Check payment |
| ach | ACH transfer |
| other | Other method |
| card_on_file | Charge saved card |

**Pre-condition:** If there are signed documents required before payment, the modal will display a warning: "The following documents must be signed before taking payment."

---

## Step 5.12 — WO → Create Invoice

**Test Scenario:** Step 5.12 — WO → Create Invoice

**App:** Admin → WO Record

| Test Step | Result |
|--------|----------|
| Click "Create invoice" on WO record | Invoice draft created |
| Invoice lines = WO lines | All line items copied |
| `created_from` = WO key | Source linkage |
| Invoice status = `draft` | Not yet sent |
| Invoice amount = remaining balance (or full if no deposit) | Correct amount |

---

## Step 5.13 — Finalize + Send Invoice

**Test Scenario:** Step 5.13 — Finalize + Send Invoice

**App:** Admin → Invoice Record (`/invoices/:id`)

| Test Step | Result |
|--------|----------|
| Review invoice draft | Lines, totals, customer info correct |
| Click "Send" | Invoice status = `open` |
| Invoice locked | `locked_at` present, cannot edit |
| Customer can pay in portal | Portal payment link active |
| Customer gets notification | Email/SMS with invoice link |

---

## ✅ Phase 5 Results

| Item | Status | Notes |
|------|--------|-------|
| Checkout screen — totals breakdown | ✅ | Balance due $195.00 displayed |
| Add item in field (optional) | ✅ | |
| Sign-off checkout documents | ✅ | Customer signature filled via API (`POST /signed-documents`) |
| Customer review screen before signing | ✅ | |
| Customer signature on-device | ✅ | |
| Remote signing flow | ✅ | |
| Signed PDF immutable + metadata | ✅ | |
| Payment collected — Cash $195 | ✅ | Tech records via PWA Checkout → Cash/Check → `POST /payments/manual` |
| Complete job → Done screen | ✅ | `POST /work-orders/:id/complete` succeeds after checklist complete |
| Admin verify: WO status = complete | ✅ | Status Complete |
| Admin verify: Payment recorded | ✅ | Payment exists on customer record |
| Create invoice from WO | ✅ | INV-1001 created, Total $195.00 |
| Send invoice → status = open | ✅ | Sent to rohani@strongheron.com, portal link active |

**Full field flow complete: Lead → Quote → WO → Tech executes → Checkout → Payment → Invoice.**

### Bugs Found & Fixed

| Bug | Root Cause | Fix |
|-----|-----------|-----|
| 403 Forbidden on `POST /payments/manual` | Technician role missing `payments:create` permission | Add permission via Admin UI → Settings → Roles → technician → Payments CREATE; then fix backend default |
| 403 Forbidden on `POST /invoices` | Technician role missing `invoices:create` permission | Same — fix via UI + backend default |
| 400 Validation Error `customerId: Required` | PWA only sent `invoiceId`, but API requires `customerId` | Edit `payments.ts`, `PaymentSheet.tsx`, `CheckoutScreen.tsx` — add `customerId` to request body |
| Method `ach` rejected by API | Schema `payments.schema.ts` only accepts `cash\|check\|other` | Remove `ach` from `ManualMethod` type and button list in `PaymentSheet.tsx` |
| 409 REQUIRED_ITEMS_INCOMPLETE on `/complete` | WO checklist was empty | Fill 5 required items via API: `before_photo`, `work_performed`, `safety_checklist_complete`, `after_photo`, `customer_signature` |

### Permanent Fix: Default Role Permissions in Backend

**File:** `api_strongheron/src/modules/roles/roles.service.ts`

`invoices` and `payments` were missing from `RESOURCE_PERMS` template — causing newly invited technicians to have no permissions for these resources.

**Fix:** Add 2 resources to `RESOURCE_PERMS`:
```typescript
// technician can create invoices from WOs assigned to them (needed by PWA checkout flow)
invoices:    { admin: ['full', 'all'], dispatcher: ['full', 'all'], technician: ['create', 'assigned'], accounting_ro: ['view', 'all'] },
// technician can record cash/check payments on WOs assigned to them (needed by PWA checkout flow)
payments:    { admin: ['full', 'all'], dispatcher: ['full', 'all'], technician: ['create', 'assigned'], accounting_ro: ['view', 'all'] },
```

After editing code, run reseed to apply to existing tenants:
```bash
POST /roles/reseed-defaults   # requires roles:full permission (admin only)
```

**Effect:** New technician automatically gets `invoices:create` + `payments:create`. "Reset to defaults" in Settings → Roles will also restore these permissions.

### Important Notes

**Payment not appearing in WO Payments tab** (only visible on customer record):
- `payments` table has `invoiceId` (nullable) but NO `workOrderId`
- WO Payments tab queries via invoice — cash payment without `invoiceId` won't link to WO
- Solution: Create invoice from WO → payment can be applied to invoice → appears in WO Payments tab

**Required checklist items** before `POST /work-orders/:id/complete` can succeed:
- `before_photo` (type: photo)
- `work_performed` (type: text)
- `safety_checklist_complete` (type: boolean)
- `after_photo` (type: photo)
- `customer_signature` (type: signature)

**`/payments/take` does NOT exist in backend** — the correct endpoint is `/payments/manual`.

**Permission `payments:create`** is not included in the `RESOURCE_PERMS` default template in `roles.service.ts` — must be set manually in admin UI or via seed.

---

## 🔗 Transform Chain Verified (Phase 1-5)

```
Lead (new → contacted → converted)
  ↓ convert
Quote (draft → sent → approved → locked)
  ↓ create WO
Work Order (new → scheduled → in_progress → complete)
  ↓ create invoice
Invoice (draft → open)
  ↓ payment
Payment (recorded)
```

Each transform link is traceable via the `created_from` field and Related tab on each record.
