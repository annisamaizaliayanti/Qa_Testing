# Phase 9 — Service Plan: Sell Subscription → Recurring Billing → Dunning

> **Concept:** Sell a maintenance plan (subscription) to a customer.
> System auto-generates visit schedule, auto-bills recurring charges, customer gets member pricing.
> If card fails → enters dunning flow. Plan can be paused, resumed, or canceled.

**Prerequisite:** Phase 1 complete. Customer with card on file (or payment method).

---

## QA Test Case Summary

| User Story | Test Scenario | Test Step | Result | Status |
|---|---|---|---|---|
| Admin sells subscription plan and system manages recurring billing | TC-9.01 Create Subscription Item | Create item type=subscription: "Gold Maintenance Plan", $100/month, 4 visits/year, memberPricing=yes | Item saved with IT_ key, subFrequency=monthly, subMemberPricing as record map |  |
| Admin sells subscription plan and system manages recurring billing | TC-9.02 Create Agreement | Create agreement via API or quote approval linking customer to subscription item | Agreement record created, status=draft → active |  |
| Admin sells subscription plan and system manages recurring billing | TC-9.03 Verify Agreement Record | Open agreement in /service-plans | Plan name, billing amount ($100), frequency (monthly), visits/year (4), start date, next bill date, status=active all shown |  |
| Admin sells subscription plan and system manages recurring billing | TC-9.04 Member Pricing Active | Create new quote for same customer | Prices resolve to Member level (e.g. Installation Labor=$400 instead of $500) |  |
| Admin sells subscription plan and system manages recurring billing | TC-9.05 Generate Occurrences | Click "Generate schedule" or POST /agreements/:id/generate-occurrences | 4 occurrences created (quarterly spread), each with due date and status=pending |  |
| Admin sells subscription plan and system manages recurring billing | TC-9.06 Create WO from Occurrence | Click "Create work order" on occurrence 1 | WO created with summary "Service visit — Gold Maintenance Plan (Visit 1)", linked to agreement via agreementId |  |
| Admin sells subscription plan and system manages recurring billing | TC-9.07 Tech Completes Visit | Tech performs normal job flow (Phase 4) on maintenance WO | WO complete, occurrence status updated to completed |  |
| Admin sells subscription plan and system manages recurring billing | TC-9.08 Auto-billing | Billing day arrives (or run POST /agreements/billing-run) | Card charged $100 (if charge_automatically), invoice + payment auto-created, next bill date advanced |  |
| Admin sells subscription plan and system manages recurring billing | TC-9.09 Card Fails → Dunning | Card charge fails (expired/declined) | Dunning state activated, retry scheduled, update-card link sent to customer |  |
| Admin sells subscription plan and system manages recurring billing | TC-9.10 Customer Updates Card | Customer opens update-card link, enters new card | Payment method replaced, saved |  |
| Admin sells subscription plan and system manages recurring billing | TC-9.11 Pause Agreement | PATCH status=paused | Status=paused (yellow badge), billing stopped, no new occurrences, "Resume" button appears |  |
| Admin sells subscription plan and system manages recurring billing | TC-9.12 Resume Agreement | PATCH status=active | Status=active (green badge), billing and occurrences resume |  |
| Admin sells subscription plan and system manages recurring billing | TC-9.13 Cancel Agreement | PATCH status=canceled | Status=canceled (grey badge), billing stopped permanently, member pricing revoked; new quote uses Standard/Preferred pricing |  |

**Reference:** [StrongHeron QA Test Cases.md](../../StrongHeron%20QA%20Test%20Cases.md) — Phase 9

---

## Step 9.01 — Create Subscription Item

**Test Scenario:** Step 9.01 — Create Subscription Item

**App:** Admin → Items → New (`/items/new`)

**Item form fields (subscription type):**

| Field | Type | Required | Value |
|-------|------|----------|-------|
| Item name | input text | ✅ | "Gold Maintenance Plan" |
| Item type | select | - | **subscription** |
| Category | select | - | Select category |
| Unit of measure | select | - | "each" |
| Cost | number | - | "50.00" (internal cost) |
| Description | textarea | - | "Annual gold maintenance plan — 4 visits/year, priority response" |
| Taxable | select | - | "no" |

**Subscription-specific fields (appear if type=subscription):**

| Field | Type | Required | Value |
|-------|------|----------|-------|
| Billing amount | number | - | "100.00" ($100/month) |
| Billing frequency | select | - | monthly / quarterly / annually → select **monthly** |
| Visits per year | number | - | "4" |
| Member pricing | select | - | **yes** |

| Test Step | Result |
|--------|----------|
| Create item with all fields → Save | Subscription item saved with `IT_` key |
| Item type = subscription | Flag active |
| Member pricing = yes | Subscribing customer gets Member price level |

---

## Step 9.02 — Sell Plan via Quote → Customer Approves

**Test Scenario:** Step 9.02 — Sell Plan via Quote → Customer Approves

**App:** Admin → Quotes → New

| Test Step | Result |
|--------|----------|
| Create quote for customer | Quote form opens |
| Add line "Gold Maintenance Plan" | Subscription item added |
| Set billing details | Amount, frequency from item |
| Send quote | Status = `sent`, customer notified |

**Customer portal:**

| Test Step | Result |
|--------|----------|
| Customer opens portal → approves + signs | Quote approved |
| **Agreement automatically created** | Service plan / agreement record |
| Agreement status = `active` | Immediately active |

---

## Step 9.03 — Check Agreement Record

**Test Scenario:** Step 9.03 — Check Agreement Record

**App:** Admin → Service Plans (`/service-plans`) → open agreement

**AgreementRecord fields displayed:**

| Field | Display |
|-------|---------|
| Plan name | "Gold Maintenance Plan" |
| Description | Plan description |
| Billing amount | $100.00 |
| Billing frequency | Monthly |
| Visits per year | 4 |
| Start date | Agreement start |
| End date | (if applicable) |
| Next bill date | Next billing cycle |
| Next visit date | Next scheduled visit |
| Status | `active` |
| Customer | Linked customer |
| Item | Linked subscription item |

**Agreement status options:**

| Status | Visual | Description |
|--------|--------|-------------|
| draft | open | Not yet active |
| active | success | Running, billing active |
| paused | warning | Temporarily stopped |
| canceled | dead | Permanently canceled |
| expired | dead | Validity period ended |

**Occurrences table:**

| Column | Description |
|--------|-------------|
| Due date | Scheduled visit date |
| Status | pending / completed / canceled |
| WO link | If WO has been created |

| Test Step | Result |
|--------|----------|
| Terms snapshot displayed | Plan details frozen |
| Billing schedule visible | Monthly billing dates |
| Occurrences seeded | Visit dates generated (4/year) |
| "Generate schedule" button | Generates occurrence dates based on visits/year |

---

## Step 9.04 — Customer Gets Member Pricing

**Test Scenario:** Step 9.04 — Customer Gets Member Pricing

**App:** Admin → Quotes → New (create new quote for same customer)

| Test Step | Result |
|--------|----------|
| Create quote for customer with active plan | Quote form opens |
| Prices resolve to **Member** level | Installation Labor = $400 (not $500 Standard or $450 Preferred) |
| Member pricing active because `member_pricing=yes` on subscription item | Auto-applied |

---

## Step 9.05 — Occurrence Due → Create WO

**Test Scenario:** Step 9.05 — Occurrence Due → Create WO

**App:** Admin → Service Plans → Agreement Record

| Test Step | Result |
|--------|----------|
| Occurrence is due | Occurrence row status = `pending` |
| Click "Create work order" on occurrence | WO created |
| WO lines = plan checklist items | Maintenance checklist |
| WO appears in dispatch | Ready to schedule |
| Occurrence status → linked to WO | WO key visible |

**Or bulk: "Run billing" button:**

| Test Step | Result |
|--------|----------|
| Click "Run billing" | Process all due occurrences |
| `res.processed` = N occurrences | Batch processed |
| WOs created for each | Multiple WOs if needed |

---

## Step 9.06 — Tech Executes Maintenance Visit

**Test Scenario:** Step 9.06 — Tech Executes Maintenance Visit

**App:** PWA (same as Phase 4)

| Test Step | Result |
|--------|----------|
| Tech views maintenance job in My Day | Visit card shown |
| Start visit → checklist → time → photos | Normal job flow |
| Complete job | WO status = complete |
| Occurrence status updated | Marked as completed |

---

## Step 9.07 — Auto-billing (Card Charged)

**Test Scenario:** Step 9.07 — Auto-billing (Card Charged)

**App:** Admin → Agreement Record

| Test Step | Result |
|--------|----------|
| Billing day arrives (based on frequency) | Auto-charge triggered |
| Card on file charged $100 | Payment processed |
| Invoice auto-created | Invoice linked to agreement |
| Payment auto-created | Payment linked to invoice |
| Next bill date updated | Advanced to next cycle |

**Collection method options (from AgreementNew):**

| Method | Description |
|--------|-------------|
| send_invoice | Send invoice, customer pays manually |
| charge_automatically | Auto-charge card on file |

---

## Step 9.08 — Card Fails → Dunning Starts

**Test Scenario:** Step 9.08 — Card Fails → Dunning Starts

**App:** Admin

| Test Step | Result |
|--------|----------|
| Card charge fails (expired/declined) | Dunning state activated |
| Retry attempt scheduled | Auto-retry logic |
| Update-card link sent to customer | Email/SMS with link |
| Agreement status remains `active` | Not immediately canceled |

**"Run dunning" button on AgreementRecord:**

| Test Step | Result |
|--------|----------|
| Click "Run dunning" | Process dunning for this agreement |
| Retry charge / send reminder | Based on dunning rules |

---

## Step 9.09 — Customer Updates Card

**Test Scenario:** Step 9.09 — Customer Updates Card

**App:** Portal → Update card link

| Test Step | Result |
|--------|----------|
| Customer opens update-card link | Card update form displayed |
| Enter new card | Card info updated |
| Card saved | Payment method replaced |

---

## Step 9.10 — Retry Success

**Test Scenario:** Step 9.10 — Retry Success

**App:** Admin

| Test Step | Result |
|--------|----------|
| Next retry attempt → new card succeeds | Payment processed |
| Dunning state → cleared | Back to current |
| Invoice paid | Balance = $0 |

---

## Step 9.11 — Pause Agreement

**Test Scenario:** Step 9.11 — Pause Agreement

**App:** Admin → Agreement Record

| Test Step | Result |
|--------|----------|
| Click "Pause" button | `handleStatusChange('paused')` |
| Agreement status = `paused` | Badge yellow "Paused" |
| No billing generated | Auto-charge stopped |
| No new occurrences | Visit schedule frozen |
| "Resume" button appears | Can reactivate |

---

## Step 9.12 — Resume Agreement

**Test Scenario:** Step 9.12 — Resume Agreement

**App:** Admin → Agreement Record

| Test Step | Result |
|--------|----------|
| Click "Resume" button | `handleStatusChange('active')` |
| Agreement status = `active` | Badge green "Active" |
| Billing resumes | Next bill date active |
| Occurrences resume | Visit schedule continues |

---

## Step 9.13 — Cancel Agreement

**Test Scenario:** Step 9.13 — Cancel Agreement

**App:** Admin → Agreement Record

| Test Step | Result |
|--------|----------|
| Click "Cancel" button (red) | `handleStatusChange('canceled')` |
| Agreement status = `canceled` | Badge grey "Canceled" |
| Billing stopped permanently | No more charges |
| Member pricing revoked | Customer no longer gets Member prices |
| Verify: create new quote → prices return to Standard/Preferred | Member discount removed |

---

## ✅ Phase 9 Results

| Item | Status | Notes |
|------|--------|-------|
| Subscription item `subFrequency: 'monthly'` | ✅ | enum must be `'monthly'` not `'yearly'` — see bugs section |
| Subscription item `subMemberPricing` as record | ✅ | must be `{ gold: 8000 }` not `true` — see bugs section |
| Create agreement directly via API | ✅ | `POST /agreements` — no need to go through quote for test |
| Agreement status draft → active | ✅ | `PATCH /agreements/:id` with `{ status: 'active' }` |
| Generate occurrences | ✅ | `POST /agreements/:id/generate-occurrences` → `{ generated: 4 }` — 4 visits quarterly |
| Create WO from occurrence | ✅ | `POST /occurrences/:id/create-work-order` → WO created |
| WO linked to agreement | ✅ | `agreementId` in WO record |
| Pause agreement | ✅ | `PATCH` status `paused` |
| Resume agreement | ✅ | `PATCH` status `active` |
| Cancel agreement | ✅ | `PATCH` status `canceled` |

**Full subscription lifecycle: Create item → Create agreement → Activate → Generate occurrences → Create WO from occurrence → Pause/Resume/Cancel.**

### Bugs Found

| Bug | Root Cause | Fix |
|-----|-----------|-----|
| 400 creating subscription item — `subFrequency: 'yearly'` | Schema enum is `'monthly' \| 'quarterly' \| 'annually'` — not `'yearly'` | Use `'monthly'` |
| 400 creating subscription item — `subMemberPricing: true` | Field is `Record<string, number>` not boolean | Use `{ gold: 8000 }` as map `memberTypeCode → priceCents` |

### Important Notes

**Agreement can be created directly** via `POST /agreements` without going through the quote flow.

**Occurrence generation:**
- `POST /agreements/:id/generate-occurrences` → generates N occurrences per `visitsPerYear`
- Occurrences spread quarterly (4 visits/year = every 3 months)
- Re-running occurrences-run will skip if already generated: `{ status: 'skipped', reason: 'already_generated' }`

**WO from occurrence:**
- `POST /occurrences/:id/create-work-order` → `{ woId }` — WO created with summary "Service visit — {planName} ({occurrenceName})"
- WO automatically linked to agreement via `agreementId` field

**Billing run only processes agreements with `nextBillDate <= today`** — if not yet due, processed = 0.

**`subMemberPricing` schema:** `z.record(z.number().int())` — key is `memberTypeCode` (e.g. `'gold'`), value is `priceCents`.

### Phase 9 Endpoint Flow

```
POST /items                                          → subscription item
POST /agreements { customerId, itemId, planName, subAmountCents, subFrequency, visitsPerYear, ... }
PATCH /agreements/:id { status: 'active' }           → activate
POST /agreements/:id/generate-occurrences            → { generated: N }
POST /occurrences/:id/create-work-order              → { woId }
PATCH /agreements/:id { status: 'paused' }           → pause
PATCH /agreements/:id { status: 'active' }           → resume
PATCH /agreements/:id { status: 'canceled' }         → cancel
POST /agreements/billing-run                         → { processed, results }
POST /agreements/occurrences-run                     → { processed, results }
POST /agreements/wo-create-run                       → { processed, results }
POST /agreements/dunning-run                         → { processed, results }
```
