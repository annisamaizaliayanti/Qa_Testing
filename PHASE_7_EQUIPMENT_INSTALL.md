# Phase 7 — Job with Equipment Install (Birth Equipment)

> **Concept:** Installing a new unit (AC, water heater, etc.) — an item in the catalog with flag `createsEquipment=true`.
> When WO completes, equipment is automatically "born" at the customer site with serial number, install date, and warranty.
> Tech must enter serial number + nameplate photo in PWA checklist before completing.

**Prerequisite:** Phase 1 complete (admin, tech, customer already set up).

---

## QA Test Case Summary

| User Story | Test Scenario | Test Step | Result | Status |
|---|---|---|---|---|
| Tech installs a new unit and system auto-creates equipment record | TC-7.00 Create Equipment Item | Create item "AC Unit" with createsEquipment=Yes, warrantyMonths=12 | Item saved; info box confirms equipment gate enabled in PWA |  |
| Tech installs a new unit and system auto-creates equipment record | TC-7.01 Create Doc Templates | Create Pre-Work Authorization (requiredAt: pre_work) and Change Authorization (requiredAt: change_auth) templates | Templates active in Settings → Checkout → Agreements |  |
| Tech installs a new unit and system auto-creates equipment record | TC-7.02 Create Install WO | Create WO with site selected, lines: AC Unit ($2,500 createsEquipment) + Installation Labor ($500) | WO created, total $3,000, site mandatory (INSTALL_REQUIRES_SITE if missing) |  |
| Tech installs a new unit and system auto-creates equipment record | TC-7.03 Schedule + In Progress | Schedule visit, set WO to in_progress | Visit scheduled, WO appears in tech's PWA My Day |  |
| Tech installs a new unit and system auto-creates equipment record | TC-7.04 Tech Starts Job | Tech opens PWA, taps job, taps "Start visit" | Visit in_progress, timer auto-starts |  |
| Tech installs a new unit and system auto-creates equipment record | TC-7.05 Pre-work Sign-off | Tap Sign-offs tile, sign pre-work authorization | authorizedCents = $3,000 (NTE ceiling set), status chip: Signed |  |
| Tech installs a new unit and system auto-creates equipment record | TC-7.06 Install Gate in Checklist | Complete regular checklist items, scroll to "⚙ New install — required fields" section | Serial number field and Nameplate photo field visible; enter SN-2024-AC-00142, upload nameplate photo; both marked ✅ |  |
| Tech installs a new unit and system auto-creates equipment record | TC-7.07 Equipment Tile Status | Tap Equipment tile on job screen | "New unit · from this job" section shows AC Unit; status changes from "No serial" (amber) to "Ready" after serial + nameplate captured |  |
| Tech installs a new unit and system auto-creates equipment record | TC-7.08 Complete Job | Complete visit → Checkout → Collect payment → Complete job | POST /work-orders/:id/complete succeeds, server auto-creates equipment record |  |
| Tech installs a new unit and system auto-creates equipment record | TC-7.09 Verify Equipment (Admin) | Open /equipment in admin | Equipment "AC Unit" with key EQP-XXXX: serial=SN-2024-AC-00142, installDate=today, warrantyExpires=today+12mo, customer + site correct |  |
| Tech installs a new unit and system auto-creates equipment record | TC-7.10 Equipment at Customer Site | Open Customer Record → Equipment tab | AC Unit listed under correct site with serial, install date, warranty filled from tech checklist |  |

**Reference:** [StrongHeron QA Test Cases.md](../../StrongHeron%20QA%20Test%20Cases.md) — Phase 7

---

## Step 7.00 — Create Item with `createsEquipment=true`

**Test Scenario:** Step 7.00 — Create Item with `createsEquipment=true`

**App:** Admin → Items (`/items`) → click **+ New item**

> **Why:** WO line item must have this flag for the install gate to activate and equipment to auto-create on complete.

| Field | Value |
|-------|-------|
| Item name | AC Unit |
| Item type | Material |
| Unit of measure | each |
| Cost ($) | (optional) |
| **Creates equipment on install** | **Yes** |
| **Warranty (months)** | **12** |
| Price (Standard) | $2,500.00 |

| Test Step | Result |
|--------|----------|
| Set "Creates equipment on install" = Yes | Info box appears: "Equipment gate enabled in PWA — Technician checklist will require serial number, nameplate photo…" |
| Fill Warranty (months) = 12 | Field set |
| Click **Create item** | Item created, redirect to Item Record |

> **Note:** `warrantyMonths=12` → server will set `warrantyExpiresOn = installDate + 12 months` automatically when WO completes.

---

## Step 7.01 — Create Doc Templates via Admin Settings

**Test Scenario:** Step 7.01 — Create Doc Templates via Admin Settings

**App:** Admin → Settings → Checkout → Agreements & Sign-offs

> **Why:** Pre-work authorization is required before tech starts work. This is a one-time setup.

### Template 1 — Pre-Work Authorization

Click **+ New template** (or Edit if it already exists):

| Field | Value |
|-------|-------|
| Name | Pre-Work Authorization |
| Kind | authorization |
| Required at | ✅ **pre_work** |
| Active | Yes |
| Body | `## Pre-Work Authorization`<br><br>`This authorizes work up to **$3,000.00** (NTE).`<br><br>`Scope: Full AC unit replacement as quoted.`<br><br>`By signing below, customer agrees to the scope and NTE amount.` |

| Test Step | Result |
|--------|----------|
| Save template | Template active, appears in list with `requiredAt: pre_work` |

### Template 2 — Change Authorization (optional — for NTE gate)

Click **+ New template**:

| Field | Value |
|-------|-------|
| Name | Change Authorization |
| Kind | authorization |
| Required at | ✅ **change_auth** |
| Active | Yes |
| Body | `## Change Authorization`<br><br>`Additional work has been identified that exceeds the original NTE.`<br><br>`By signing, customer authorizes the additional costs.` |

| Test Step | Result |
|--------|----------|
| Save template | Template active with `requiredAt: change_auth` |

---

## Step 7.02 — Create Install WO

**Test Scenario:** Step 7.02 — Create Install WO

**App:** Admin → Work Orders → New (`/work-orders/new`)

| Field | Value |
|-------|-------|
| Customer | Select existing customer |
| **Site / location** | **Must select a site** (no site → error `INSTALL_REQUIRES_SITE` on complete) |
| Job type | AC Install (or Installation) |
| Assigned tech | Select tech |
| Summary | "Install new AC unit — Carrier 48TC060" |

**Line items:**

| Item | Qty | Rate | creates_equipment |
|------|-----|------|-------------------|
| AC Unit | 1 | $2,500.00 | ✅ yes (automatic from item flag) |
| Installation Labor | 1 | $500.00 | no |

| Test Step | Result |
|--------|----------|
| Select customer → select site | Site dropdown populated from customer |
| Add line "AC Unit" | Rate auto-fills $2,500.00 |
| Add line "Installation Labor" | Rate auto-fills $500.00 |
| Click **Create work order** | WO created, redirect to WO Record |
| WO Record total | $3,000.00 |

---

## Step 7.03 — Schedule Visit + Set In Progress

**Test Scenario:** Step 7.03 — Schedule Visit + Set In Progress

**App:** Admin → WO Record → **Visits** tab

### Schedule visit:

| Test Step | Result |
|--------|----------|
| Click **Visits** tab | Visits tab displayed |
| Click **+ Schedule a visit** | Schedule form appears |
| Fill date + assigned tech → Save | Visit created with status `scheduled` |

### Set WO to In Progress:

| Test Step | Result |
|--------|----------|
| On WO Record → click status badge (top right) | Status dropdown appears |
| Select **In Progress** | WO status changes to `in_progress` |
| WO appears in PWA My Day for tech | Job shown in tech's dashboard |

### Set Visit to In Progress (alternative):

| Test Step | Result |
|--------|----------|
| Visits tab → click **Start** button next to visit | Visit status → `in_progress` |

---

## Step 7.04 — Tech Starts Job

**Test Scenario:** Step 7.04 — Tech Starts Job

**App:** PWA → Login as tech → `/today` (My Day)

| Test Step | Result |
|--------|----------|
| Tech opens PWA, logs in | Tech dashboard displayed |
| WO appears in My Day | Job card shown with customer name + summary |
| Tap job → open Job Screen | Job screen displayed |
| Tap **Start visit** | Visit status → `in_progress`, navigate to Checklist |
| Timer auto-starts | Clock icon active |

---

## Step 7.05 — Pre-Work Sign-off (NTE Capture)

**Test Scenario:** Step 7.05 — Pre-Work Sign-off (NTE Capture)

**App:** PWA → Job Screen → **Sign-offs** tile

> **This is where NTE is captured.** After signing, `authorizedCents` = `totalCents` at that time ($3,000.00).

| Test Step | Result |
|--------|----------|
| Tap **Sign-offs** tile | Sign-offs screen displayed |
| "Pre-Work Authorization" appears as pending | Status chip: Awaiting |
| Tap row → navigate to review screen | Document body displayed (NTE $3,000.00) |
| Return → signature capture shown | Signature canvas displayed |
| Enter signer name → Sign → Submit | Signed document created |
| `authorizedCents` = 300,000 cents ($3,000.00) | NTE ceiling set |
| Status chip changes to "Signed" |  |

---

## Step 7.06 — Fill Checklist

**Test Scenario:** Step 7.06 — Fill Checklist

**App:** PWA → Job Screen → **Checklist** tile

| Test Step | Result |
|--------|----------|
| Tap **Checklist** tile | Checklist screen displayed |
| Complete regular items (checkbox, photo, etc.) | Items marked done |
| **Scroll down** — section "⚙ New install — required fields" appears | This section appears automatically because WO has a line with `createsEquipment=true` |

### Install gate items (section "⚙ New install — required fields"):

| Item | How to fill | Expected |
|------|----------|----------|
| **Serial number (from nameplate)** | Type serial from unit nameplate: `SN-2024-AC-00142` → tap **Save** | Status changes to ✅ |
| **Nameplate photo** | Tap **Add** → camera opens → take photo of nameplate | Photo saved, thumbnail shown |

| Test Step | Result |
|--------|----------|
| Both install gate items done | Progress bar: "All required done" |
| "Complete visit" button active (not disabled) | Gate cleared |

---

## Step 7.07 — Equipment Tile — Status Update

**Test Scenario:** Step 7.07 — Equipment Tile — Status Update

**App:** PWA → Job Screen → **Equipment** tile

| Test Step | Result |
|--------|----------|
| Tap **Equipment** tile on Job Screen | Equipment screen displayed |
| **"Units at this site"** section | Existing units at customer site (if any) |
| **"New unit · from this job"** section | "AC Unit" from WO line shown |
| Before serial filled | Status chip: `[No serial]`, "Go to Checklist" button shown |
| After serial + nameplate filled (Step 7.06) | Status chip: `[Ready]`, subtitle: "Serial + nameplate captured — ready for completion" |

---

## Step 7.08 — Complete Visit → Checkout → Complete Job

**Test Scenario:** Step 7.08 — Complete Visit → Checkout → Complete Job

**App:** PWA → Checklist → Complete visit → Checkout

| Test Step | Result |
|--------|----------|
| Tap **Complete visit** in Checklist | Visit status → `done`, navigate to Checkout |
| Checkout screen displayed | Total $3,000.00 |
| Collect payment (cash/card/invoice) | Payment recorded |
| Tap **Complete job** | `POST /work-orders/:id/complete` |
| **Server auto-creates equipment** | Equipment record created automatically |
| Navigate to Done screen | "Job Complete" or "Paid $X" |

**Server-side on complete:**
```
Detects: WO line "AC Unit" has createsEquipment = true
→ Creates equipment record:
   name              = "AC Unit"
   serial            = "SN-2024-AC-00142"   (from install_serial response)
   installDate       = today (2026-10-21)
   warrantyExpiresOn = installDate + 12 months = 2027-10-21
   customerId        = customer from WO
   siteId            = site from WO (REQUIRED)
   source            = 'installed_by_us'
   sourceWoId        = ID of this WO
   nameplateFileId   = file from install_nameplate_photo response
```

---

## Step 7.09 — Verify Equipment Created (Admin)

**Test Scenario:** Step 7.09 — Verify Equipment Created (Admin)

**App:** Admin → Equipment (`/equipment`)

| Test Step | Result |
|--------|----------|
| Open Equipment page | Equipment list displayed |
| New equipment appears | "AC Unit" with key EQP-XXXX |
| Click equipment record | Details displayed |

**Auto-populated fields:**

| Field | Expected Value |
|-------|----------------|
| Name | AC Unit |
| Serial | SN-2024-AC-00142 |
| Install date | 2026-10-21 |
| Warranty expires | 2027-10-21 |
| Customer | Customer from WO |
| Site | Site from WO |

---

## Step 7.10 — Verify Equipment at Customer Site

**Test Scenario:** Step 7.10 — Verify Equipment at Customer Site

**App:** Admin → Customers → select customer → **Equipment** tab

| Test Step | Result |
|--------|----------|
| Open Customer Record | Equipment tab present |
| Equipment tab | "AC Unit" unit listed |
| Unit listed under correct site | Not under another site |
| Serial, install date, warranty filled | Complete data from tech checklist |

---

## ✅ Phase 7 Results

| Item | Status | Notes |
|------|--------|-------|
| Item "AC Unit" with `createsEquipment=true`, `warrantyMonths=12` | ✅ | Via Admin → Items → New Item |
| Doc template "Pre-Work Authorization" (`requiredAt: pre_work`) | ✅ | Via Admin → Settings → Checkout → Agreements |
| WO created with site + line "AC Unit" | ✅ | Site must be selected |
| Visit scheduled + status `in_progress` | ✅ | Via Admin WO Record (status dropdown + Visits tab) |
| Tech starts job in PWA | ✅ | Tap "Start visit" |
| Pre-work sign-off → `authorizedCents` = $3,000 | ✅ | Sign-offs tile in PWA |
| Checklist: install gate — serial + nameplate photo | ✅ | Section "⚙ New install" in Checklist screen |
| Equipment tile shows "New unit · from this job" | ✅ | Status "No serial" → "Ready" after serial entered |
| Complete visit → Complete job | ✅ | Checklist → Checkout → Complete |
| Equipment auto-created when WO completes | ✅ | `createsEquipment` trigger on server |
| Equipment: serial, install date, warranty | ✅ | Data from tech checklist |
| Equipment registered at customer site | ✅ | Verified via Admin → Equipment / Customer |

---

## Technical Reference

### Install Gate Items (PWA `ChecklistScreen.tsx`)

```typescript
// Constant in ChecklistScreen.tsx — 2 extra items that appear automatically
const INSTALL_GATE_ITEMS: FormItem[] = [
  { key: 'install_serial',          label: 'Serial number (from nameplate)', type: 'text',  required: true },
  { key: 'install_nameplate_photo', label: 'Nameplate photo',                type: 'photo', required: true },
]

// Trigger: WO line with createsEquipment = true
const hasInstallLine = (workOrder?.lines ?? []).some(l => l.createsEquipment)
// → section "⚙ New install — required fields" appears below checklist
```

### Equipment Screen — Two Sections

```
EquipmentScreen.tsx:

Section 1: "Units at this site"
  → listEquipment({ siteId }) — existing equipment already at site

Section 2: "New unit · from this job"  (only appears if installLines exist)
  → workOrder.lines.filter(l => l.createsEquipment)
  → Status: "No serial" (amber) or "Ready" (ok) depending on form responses
  → "Go to Checklist" button if serial not yet entered
```

### Equipment Auto-Birth Logic (server on `/complete`)

```
for each WO line with item.createsEquipment = true:
  → create equipment record:
     name              = item.name
     serial            = responses['install_serial'].value
     installDate       = today
     warrantyExpiresOn = installDate + item.warrantyMonths months
     siteId            = wo.siteId  (REQUIRED — error INSTALL_REQUIRES_SITE if missing)
     source            = 'installed_by_us'
     sourceWorkOrderId = wo.id
     nameplateFileId   = responses['install_nameplate_photo'].value
```

### Two Ways to Create Equipment — Difference

| | Via WO Install (`createsEquipment=true`) | Via Admin Manual (`/equipment/new`) |
|--|--|--|
| Who enters | Tech in field via PWA checklist | Admin in office |
| When | Automatically on WO complete | Any time |
| Install date | Automatically = date of completion | Admin enters manually |
| Warranty | Automatically = installDate + warrantyMonths | Admin calculates + enters manually |
| Serial | Tech scans/types from nameplate | Admin types |
| Nameplate photo | ✅ Saved from checklist | ❌ Not available |
| Linked to WO | ✅ `sourceWorkOrderId` | ❌ Not linked |

### Error Codes

| Code | Cause | Fix |
|------|---------|-----|
| `INSTALL_REQUIRES_SITE` | WO has no `siteId` | Create WO with site selected |
| `REQUIRED_ITEMS_INCOMPLETE` | `install_serial` or `install_nameplate_photo` not filled | Complete in Checklist → section "⚙ New install" |
| `SIGN_OFFS_REQUIRED` | Pre-work sign-off not signed | Go to Sign-offs tile first |

### Bugs Found & Fixed (Historical)

| Bug | Root Cause | Fix |
|-----|-----------|-----|
| 400 when creating WO line — `unitPriceCents` not recognized | API field is `rateCents` | Use `rateCents` |
| 400 when submitting form responses — `responses` expected object not array | Schema expects `Record<key, {value}>` | Send as object |
| 409 `INSTALL_REQUIRES_SITE` on complete | Equipment birth requires site | Set `siteId` on WO at creation |
| 409 `SIGN_OFFS_REQUIRED` on start visit | Template has `requiredAt: ['pre_work']` | Sign pre-work sign-off first |
