# Phase 6 — Job with Deposit Gate + Sign-off + NTE

> **Concept:** An expensive job that requires a deposit before tech can start work.
> Pre-work authorization captures the NTE (Not-To-Exceed) via signature.
> If the total exceeds NTE while in the field, tech must request a change authorization before checking out.

**Prerequisite:** Phase 1 complete (admin, tech, items, checklists). Customer already exists.

> **Important note about NTE / authorizedCents:**
> - Field `nteCents` does NOT exist in the database — the actual field is `authorizedCents`
> - `authorizedCents` is set **automatically** when the pre-work sign-off is signed (not from form input)
> - The NTE value written in the pre-work document becomes `authorizedCents` after signing
> - Fields **Deposit Required** and **Deposit Amount** do NOT exist in the "New Work Order" form
>   → Set via UI on the WO Record after the WO is created (DEPOSIT section in right panel, see Step 6.01b)

---

## QA Test Case Summary

| User Story | Test Scenario | Test Step | Result | Status |
|---|---|---|---|---|
| System blocks tech from starting until deposit paid and NTE set | TC-6.00 Prepare Catalog Items | Create 3 items: AC Unit Full Replacement ($2,500), Labor Installation ($400), Refrigerant R-410A ($300) | All 3 items saved in catalog |  |
| System blocks tech from starting until deposit paid and NTE set | TC-6.01 Create WO with Lines | Create WO with AC Unit + Labor lines ($2,900 total) | WO created, subtotalCents=290000, totalCents=290000 |  |
| System blocks tech from starting until deposit paid and NTE set | TC-6.01b Set Deposit via UI | On WO Record panel, toggle Required=Yes, set Amount=$500 | Deposit section shows Required: Yes, Amount: $500.00, "Create & send deposit invoice" button visible |  |
| System blocks tech from starting until deposit paid and NTE set | TC-6.02 Schedule + Set In Progress | Schedule visit, set WO status to in_progress | Visit created, WO visible in tech's PWA My Day |  |
| System blocks tech from starting until deposit paid and NTE set | TC-6.03 Deposit Gate — Tech Blocked | Tech opens job in PWA | Banner: "Deposit $500.00 required — not paid. Contact office.", depositRequired && !depositPaid = true |  |
| System blocks tech from starting until deposit paid and NTE set | TC-6.04 Start Button Hidden | Observe job screen | "Start visit" button NOT shown, replaced by deposit gate warning |  |
| System blocks tech from starting until deposit paid and NTE set | TC-6.05 Create & Send Deposit Invoice | Admin clicks "Create & send deposit invoice" on WO | Deposit invoice created (INV-XXXX), linked to WO deposit section |  |
| System blocks tech from starting until deposit paid and NTE set | TC-6.06 Pay Deposit | Admin records $500 payment against deposit invoice (via Take Payment or API) | depositInvoiceStatus=paid, WO shows "Paid: ✓ Paid" green badge, depositPaid=true |  |
| System blocks tech from starting until deposit paid and NTE set | TC-6.07 Gate Cleared — Tech Starts | Tech refreshes job in PWA | Deposit gate gone, "Start visit" button appears, tap starts visit + timer |  |
| System blocks tech from starting until deposit paid and NTE set | TC-6.08 Pre-work Sign-off (NTE) | Tech navigates to Sign-offs, signs pre-work authorization document | authorizedCents set to totalCents ($2,900), NTE ceiling captured |  |
| System blocks tech from starting until deposit paid and NTE set | TC-6.09 Tech Works Job | Complete checklist, timer, photos | Normal Phase 4 flow, tiles updated |  |
| System blocks tech from starting until deposit paid and NTE set | TC-6.10 Total Exceeds NTE | Add Refrigerant R-410A ($300) in Materials, total rises to $3,200 | Warning banner: "⚠ Change auth needed — +$300.00", overNte=true (totalCents > authorizedCents) |  |
| System blocks tech from starting until deposit paid and NTE set | TC-6.11 Checkout Blocked | Tech tries to open checkout | NTE warning: "Costs exceed authorized amount", delta +$300 shown, checkout disabled |  |
| System blocks tech from starting until deposit paid and NTE set | TC-6.12 Change Authorization | Tech navigates to Sign-offs, change_auth document shown, customer signs | authorizedCents raised to $3,200 (matches totalCents), over-NTE gate cleared |  |
| System blocks tech from starting until deposit paid and NTE set | TC-6.13 Admin Override (Alt) | Admin PATCHes authorizedCents=320000 with authorizationNote | authorizedCents = totalCents, gate cleared, tech can proceed |  |
| System blocks tech from starting until deposit paid and NTE set | TC-6.14 Signed PDF Immutable | Admin opens signed pre-work auth document | PDF shows original content at signing, locked_at timestamp, metadata (signer, signedAt, method) present; editing WO does not alter PDF |  |
| System blocks tech from starting until deposit paid and NTE set | TC-6.15 Checkout Succeeds | Tech opens checkout | NTE warning gone, totals show $3,200, balance = $3,200 - $500 deposit = $2,700, complete job succeeds |  |

**Reference:** [StrongHeron QA Test Cases.md](../../StrongHeron%20QA%20Test%20Cases.md) — Phase 6

---

## Step 6.00 — Prepare: Create Items in Catalog

**Test Scenario:** Step 6.00 — Prepare: Create Items in Catalog

> **Why:** WO needs line items to have `totalCents > 0`. Without items, cannot trigger NTE warning.

**App:** Admin → Items (`/items`) → New item

Create 3 items one by one:

| Item | Unit Price | Used in |
|------|-----------|----------------|
| AC Unit - Full Replacement | $2,500.00 | WO line item |
| Labor - Installation | $400.00 | WO line item |
| Refrigerant R-410A | $300.00 | Added in field to trigger NTE |

**How to create item:**
1. Admin → click **Items** in sidebar → click **+ New item**
2. Fill Name, set price
3. Save

**After 3 items created:** Catalog ready to use when creating WO.

---

 **Create work order** | WO created, redirect to `/work-orders/:id` |
| WO Record displayed with total $2,900.00 | `subtotalCents=290000`, `totalCents=290000` |

---

## Step 6.01b — Set Deposit on WO Record

**Test Scenario:** Step 6.01b — Set Deposit on WO Record

> **Can now be done via UI directly** — DEPOSIT section in WO Record right panel with editable fields.

**App:** Admin → Newly created WO Record → right panel

| Test Step | Result |
|--------|----------|
| Scroll to **DEPOSIT** section in right panel | "Required: No ✎" displayed (always visible) |
| Click **No** in Required row | **No / Yes** toggle appears |
| Click **Yes** | Auto-saves, page reloads |
| **Amount** row appears with "— Set amount ✎" | Amount field editable |
| Click **— Set amount** | Number input with `$` prefix opens |
| Type `500` → press Enter or click ✓ | Saves, page reloads |
| WO Record after reload | **Required: Yes ✎ · Amount: $500.00 ✎ · Invoice: — · Paid: Unpaid** |
| **Create & send deposit invoice** button appears | Ready to create |

> **Note:** If the WO already has a deposit invoice, the Required and Amount fields become read-only (cannot be changed) to prevent inconsistency.

---

## Step 6.02 — Schedule Visit + Set Status In Progress

**Test Scenario:** Step 6.02 — Schedule Visit + Set Status In Progress

**App:** Admin → WO Record → Visits tab

> **Why:** Tech can only see the job in PWA if there is a scheduled/in_progress visit assigned to them.

| Test Step | Result |
|--------|----------|
| Click **Visits** tab | Visits tab displayed |
| Click **+ Schedule a visit** | Schedule form appears |
| Fill date + assigned tech → Save | Visit created with status `scheduled` |

**Set WO status to `in_progress` via API (required for visit to appear in PWA):**
```bash
curl -X PATCH http://localhost:3000/work-orders/{WO_ID} \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {TOKEN}" \
  -d '{"status": "in_progress"}'
```

| Test Step | Result |
|--------|----------|
| PATCH status → `in_progress` | WO appears in tech's "My Day" in PWA |

---

## Step 6.03 — Tech Views Job — Deposit Blocked

**Test Scenario:** Step 6.03 — Tech Views Job — Deposit Blocked

**App:** PWA → Login as tech → `/today` (My Day)

| Test Step | Result |
|--------|----------|
| Tech opens PWA, logs in | Tech dashboard displayed |
| WO appears in My Day | Job card shown |
| Tap job → open `/job/:workOrderId` | Job screen displayed |
| **Deposit gate active** | Banner: "Deposit $500.00 required — not paid. Contact office." |
| `wo.depositRequired && !wo.depositPaid` → true | Gate logic in `JobScreen.tsx` |

---

## Step 6.04 — Tech Cannot Start

**Test Scenario:** Step 6.04 — Tech Cannot Start

**App:** PWA → Job Screen

| Test Step | Result |
|--------|----------|
| **"Start visit"** button NOT shown | Replaced by deposit gate warning |
| Tech cannot start work | Must wait for deposit payment |
| Tap "Message office" (if available) | Directs to chat/message admin |

---

## Step 6.05 — Admin Creates & Sends Deposit Invo## Step 6.01 — Create WO with Line Items

**Test Scenario:** Step 6.01 — Create WO with Line Items

**App:** Admin → Work Orders → New (`/work-orders/new`)

> **IMPORTANT:** The "New Work Order" form does NOT have NTE amount, Deposit Required, or Deposit Amount fields.
> These fields are set **after** the WO is created via API PATCH (see Step 6.01b).

**WO form fields available in UI:**

| Field | Type | Value |
|-------|------|-------|
| Customer | select | Select existing customer (e.g. Sarah TC1) |
| Site / location | select | Select customer site |
| Assigned tech | select | Select tech (e.g. technical 1 TC) |
| Job type | select | "Installation" (or per available value set) |
| Priority | select | "Low" |
| Price list | select | "Standard price" |
| Summary | textarea | "Full AC replacement — expensive job, deposit required" |
| Internal notes | textarea | (leave blank) |
| Date (Schedule) | date | Fill today's date (optional) |

**Line Items — add 2 items from catalog:**

| Item | Qty | Rate | Total |
|------|-----|------|-------|
| AC Unit - Full Replacement | 1 | $2,500.00 | $2,500.00 |
| Labor - Installation | 1 | $400.00 | $400.00 |

> **Subtotal after 2 items: $2,900.00**
> Third item (Refrigerant $300) will be added in the field later to trigger NTE warning.

| Test Step | Result |
|--------|----------|
| Select customer + site | Form shows site options |
| Fill job details | Fields filled |
| Add line item: select "AC Unit - Full Replacement" | Rate auto-fills $2,500.00 |
| Add second line item: "Labor - Installation" | Rate auto-fills $400.00 |
| Clickice

**Test Scenario:** Step 6.05 — Admin Creates & Sends Deposit Invoice

**App:** Admin → WO Record

| Test Step | Result |
|--------|----------|
| Open WO Record → right panel DEPOSIT section | "Required: $500.00" displayed |
| Click **Create & send deposit invoice** | Confirmation modal appears |
| Confirm → Submit | Deposit invoice created (INV-XXXX) |
| Invoice key shown on WO Record | "Invoice: INV-XXXX", "Paid: Unpaid" |

---

## Step 6.06 — Admin Processes Deposit Payment

**Test Scenario:** Step 6.06 — Admin Processes Deposit Payment

**App:** Admin → WO Record → Take Payment

> **IMPORTANT:** A regular payment (without `invoiceId`) will NOT change `depositPaid`.
> Must pay **via the deposit invoice** created in Step 6.05.

**Method 1 — Via Take Payment on WO Record:**

| Field | Value |
|-------|-------|
| CUSTOMER | Auto-selected |
| PAYMENT METHOD | Cash / Check / Other |
| AMOUNT ($) | 500.00 |
| APPLY TO INVOICE | Select deposit invoice (INV-XXXX) |
| NOTE | "Deposit payment received" |

**Method 2 — Via API manual payment:**
```bash
# Get deposit invoice ID from WO response field depositInvoiceId
curl -X POST http://localhost:3000/payments/manual \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {TOKEN}" \
  -d '{
    "customerId": "{CUSTOMER_ID}",
    "invoiceId": "{DEPOSIT_INVOICE_ID}",
    "amountCents": 50000,
    "method": "cash",
    "note": "Deposit payment received"
  }'
```

| Test Step | Result |
|--------|----------|
| Payment recorded to deposit invoice | `depositInvoiceStatus` → `paid` |
| WO Record → DEPOSIT section | "Paid: ✓ Paid" (green badge) |
| `wo.depositPaid` = true | Gate will clear in PWA |

---

## Step 6.07 — Tech Can Now Start

**Test Scenario:** Step 6.07 — Tech Can Now Start

**App:** PWA → Job Screen

| Test Step | Result |
|--------|----------|
| Tech refreshes / re-opens job | Deposit gate gone |
| `wo.depositPaid = true` | Gate passed |
| **"Start visit" button appears** | Tech can begin |
| Tap **Start visit** | Visit status → `in_progress`, navigate to time screen |
| Timer starts running | Clock icon active |

---

## Step 6.08 — Pre-work Sign-off Required (NTE Capture)

**Test Scenario:** Step 6.08 — Pre-work Sign-off Required (NTE Capture)

**App:** PWA → Sign-off Screen (`/job/:id/signoff`)

> **This is where NTE is captured.** The pre-work authorization document contains the NTE amount.
> After signing, `authorizedCents` = the agreed value in the document.

**Prerequisite — Create pre-work doc template in Admin via API:**
```bash
# Create doc template type pre_work
curl -X POST http://localhost:3000/doc-templates \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {TOKEN}" \
  -d '{
    "name": "Pre-Work Authorization",
    "requiredAt": ["pre_work"],
    "bodyMarkdown": "## Pre-Work Authorization\n\nThis authorizes work up to **$2,900.00** (NTE).\n\nScope: Full AC replacement as quoted.\n\nBy signing below, customer agrees to the scope and NTE amount."
  }'
```

| Test Step | Result |
|--------|----------|
| Doc template created | `id` = new template ID |
| Navigate to `/job/:id/signoff` in PWA | Sign-off screen displayed |
| Pre-work authorization document shown | Body contains NTE $2,900.00 |
| Tech + customer sign | Signature canvas shown |
| Submit signatures | Signed document record created |
| `authorizedCents` set = `totalCents` at that time ($2,900.00) | NTE ceiling = $290,000 cents |

---

## Step 6.09 — Tech Works Job

**Test Scenario:** Step 6.09 — Tech Works Job

**App:** PWA (same as Phase 4-5)

| Test Step | Result |
|--------|----------|
| Fill checklist items | Required items done |
| Record time (timer or manual) | Time entry created in PWA |
| Take photos | Before/after photos |
| Tile counts update | Progress shown on job screen |

> **Note on Time Entries — PWA vs Admin:**
> - Time recorded in PWA (`/job/:id/time`) **appears immediately in PWA** but does **NOT immediately appear** in Admin WO → Time tab
> - Admin WO Time tab only shows entries that have been **approved** through the timesheet workflow
>
> **Full flow for time to appear in Admin:**
> 1. Tech records time in PWA → saved as time entry (status: draft)
> 2. Tech opens `/me/timesheet` in PWA → clicks **Submit for approval**
> 3. Admin/manager approves timesheet
> 4. After approval → entries appear in Admin WO → Time tab with hours and cost code

---

## Step 6.10 — Total Exceeds NTE → Warning Appears

**Test Scenario:** Step 6.10 — Total Exceeds NTE → Warning Appears

**App:** PWA → Materials Screen (`/job/:id/materials`)

> Add the third item (**Refrigerant R-410A, $300**) to make total > `authorizedCents` ($2,900).
> New total = $3,200 → exceeds `authorizedCents` $2,900 → delta +$300.

| Test Step | Result |
|--------|----------|
| Open Materials tab | Materials/line items list displayed |
| Tap **+ Add material** | Modal or form to add item |
| Select "Refrigerant R-410A", qty 1, $300.00 | Item added to WO |
| Total rises to $3,200 | `totalCents` = 320,000 |
| Job screen HeroCard updates | **Warning banner**: "⚠ Change auth needed — +$300.00" |
| `wo.authorizedCents` ($290,000) < `wo.totalCents` ($320,000) | `overNte = true` |

**In Admin WO Record (right panel TOTALS):**
```
Authorized: $2,900.00  ⚠ Change Auth +$300.00
```

---

## Step 6.11 — Checkout Blocked Due to Over-NTE

**Test Scenario:** Step 6.11 — Checkout Blocked Due to Over-NTE

**App:** PWA → Checkout Screen (`/job/:id/checkout`)

| Test Step | Result |
|--------|----------|
| Tech tries to open checkout | **NTE warning shown** |
| "Costs exceed authorized amount" banner | Delta +$300.00 displayed |
| Checkout **cannot proceed** | Button disabled or blocked |
| `wo.authorizedCents > 0 && wo.totalCents > wo.authorizedCents` | Over-NTE gate active |

---

## Step 6.12 — Change Authorization (Delta Auth)

**Test Scenario:** Step 6.12 — Change Authorization (Delta Auth)

**App:** PWA → Sign-offs Screen (`/job/:id/signoff`)

> Create change_auth doc template in Admin first (one-time setup):

```bash
curl -X POST http://localhost:3000/doc-templates \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {TOKEN}" \
  -d '{
    "name": "Change Authorization",
    "requiredAt": ["change_auth"],
    "bodyMarkdown": "## Change Authorization\n\nAdditional work has been identified that exceeds the original NTE.\n\nAdditional amount: to be determined at time of signing.\n\nBy signing, customer authorizes the additional costs."
  }'
```

| Test Step | Result |
|--------|----------|
| Navigate to Sign-offs in PWA | Change auth doc shown (pending) |
| Body shows delta amount (+$300.00) | Auto-calculated from `totalCents - authorizedCents` |
| Customer signs | Signature captured |
| Submit → signed document created | `authorizedCents` raised to `totalCents` ($3,200.00) |
| `authorizedCents` ≥ `totalCents` | Over-NTE gate cleared |

> **Note:** Change auth is considered "signed" based on `authorizedCents >= totalCents`, not from signed document ID check — different from pre_work/checkout sign-offs.

---

## Step 6.13 — Admin Override NTE Gate (Alternative)

**Test Scenario:** Step 6.13 — Admin Override NTE Gate (Alternative)

> **Purpose:** If customer is unavailable to sign, admin can override NTE gate by providing a reason.

**App:** Admin → WO Record

```bash
# Via API — set authorizedCents directly to total (override)
curl -X PATCH http://localhost:3000/work-orders/{WO_ID} \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer {TOKEN}" \
  -d '{
    "authorizedCents": 320000,
    "authorizationNote": "Customer approved via phone call — John Smith 10/21/2026"
  }'
```

| Test Step | Result |
|--------|----------|
| `authorizedCents` = `totalCents` | Over-NTE gate cleared |
| WO Record shows authorized = $3,200.00 | No warning |
| Tech can proceed to checkout | Gate passed |

---

## Step 6.14 — Signed PDF Immutable After Pre-work Auth

**Test Scenario:** Step 6.14 — Signed PDF Immutable After Pre-work Auth

> **Purpose:** Verify that the signed PDF from pre-work authorization is locked and does not change even if admin edits WO data afterward.

**App:** Admin → WO Record

| Test Step | Result |
|--------|----------|
| Open WO → view signed documents | Signed pre-work auth shown |
| Try editing WO field (summary, notes) | Field can be changed |
| Open signed pre-work auth PDF | PDF still shows **ORIGINAL** content at signing time |
| `locked_at` timestamp present in doc record | Document immutable — cannot be regenerated |
| `signerName`, `signedAt`, `method` present in metadata | Complete audit trail |

---

## Step 6.15 — Checkout Now Succeeds

**Test Scenario:** Step 6.15 — Checkout Now Succeeds

**App:** PWA → Checkout Screen

| Test Step | Result |
|--------|----------|
| NTE warning gone | `authorizedCents >= totalCents` |
| Normal checkout flow | Lines + totals displayed ($3,200.00) |
| Sign-off checkout documents (if any) | Collect customer signature |
| Collect payment | Cash/check/card — remaining balance after deposit |
| **Balance = $3,200 - $500 deposit = $2,700** | Collect $2,700 or as agreed |
| Click **Complete** | Done screen appears |
| WO status → `complete` | `POST /work-orders/:id/complete` succeeds |

---

## ✅ Phase 6 Results

| Item | Status | Notes |
|------|--------|-------|
| Create 3 items in catalog | ✅ | AC Unit $2,500 · Labor $400 · Refrigerant $300 |
| WO created with 2 line items ($2,900) | ✅ | Deposit set via PATCH API |
| Deposit gate in PWA | ✅ | `depositRequired=true`, tech blocked |
| Admin creates deposit invoice → pays | ✅ | `depositPaid=true` → gate cleared |
| Pre-work sign-off → NTE captured ($2,900) | ✅ | `authorizedCents=290000` |
| Tech works job | ✅ | Checklist, timer, photos |
| Add $300 item → total > NTE | ✅ | `totalCents=320000 > authorizedCents=290000` |
| Change auth signed → NTE raised to $3,200 | ✅ | `authorizedCents=320000` |
| Checkout succeeds — balance $2,700 | ✅ | After $500 deposit deducted |

---

## Technical Reference

### Fields that EXIST in DB (`work_orders` table)

| Column | Type | Notes |
|--------|------|-------|
| `deposit_required` | boolean | Default false |
| `deposit_amount_cents` | int | Default 0 |
| `deposit_invoice_id` | text | FK to invoices |
| `authorized_cents` | int | Auto-set from sign-off |
| `total_cents` | int | Subtotal + tax |
| `paid_cents` | int | Total paid |
| `balance_cents` | int | total - deposit - paid |

### Fields NOT in New WO form (set after WO created)
- `depositRequired` → set via UI: DEPOSIT section in WO Record right panel → click Required toggle
- `depositAmountCents` → set via UI: DEPOSIT section in WO Record right panel → click Amount field
- `authorizedCents` → automatically set from signed pre-work document

### NTE Logic (in WorkOrderRecord.jsx)
```js
const authorizedCents = wo?.authorizedCents ?? 0
const overNte = authorizedCents > 0 && totalCents > authorizedCents
// → "Change Auth +$X" warning when overNte = true
```

### Deposit Gate Logic (in JobScreen.tsx)
```js
const depositBlocked = wo.depositRequired && !wo.depositPaid
// → "Start visit" not shown when depositBlocked = true
```

### `authorizedCents` Flow
```
pre_work signed → authorizedCents = wo.totalCents (if totalCents > authorizedCents or authorizedCents = 0)
change_auth signed → authorizedCents = wo.totalCents (if authorizedCents < totalCents)
```

### Permanent Fix: Doc Template `externalRef` Auto-Generate
**Root cause:** DB constraint `UNIQUE NULLS NOT DISTINCT (tenant_id, external_ref)` treats all `NULL` as the same value.
**Fix:** `doc-templates.service.ts` → `createDocTemplate` auto-generates `externalRef = ulid()` if not provided — every template gets a unique ID automatically.
