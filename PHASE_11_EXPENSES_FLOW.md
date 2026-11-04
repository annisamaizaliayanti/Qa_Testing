# Phase 11 — Expenses Flow: Tech Records → Submits → Admin Approves → Bills to Customer

> **Concept:** Tech purchases materials in the field, records the expense in the PWA, submits for approval.
> Admin reviews, approves (or rejects), then bills billable expenses to the customer via invoice.

**Prerequisite:** An active WO assigned to tech exists (Phase 2/3/4).

---

## QA Test Case Summary

| User Story | Test Scenario | Test Step | Result | Status |
|---|---|---|---|---|
| Tech records expense and admin approves for billing to customer | TC-11.01 Tech Creates Expense | Open /job/:id/expenses, tap "+ Add Expense", fill category, vendor ("Home Depot"), description, amount ($157), date, billable=yes | Expense created, status=draft, appears in list with amount/vendor/billable badge |  |
| Tech records expense and admin approves for billing to customer | TC-11.02 Tech Submits Expense | Tap "Submit" on draft expense | Status: draft → submitted (blue badge), expense locked from editing, appears in admin queue |  |
| Tech records expense and admin approves for billing to customer | TC-11.03 Admin Views Pending | Open /expenses, filter status=submitted | Submitted expenses listed with KEY, DATE, MEMBER, CATEGORY, VENDOR, AMOUNT, BILLABLE, STATUS columns |  |
| Tech records expense and admin approves for billing to customer | TC-11.04 Admin Approves | Click "Approve" on expense | Status: submitted → approved (green badge), cost posted to WO, appears in WO T&M Unbilled section |  |
| Tech records expense and admin approves for billing to customer | TC-11.05 Unbilled Indicator | Open WO → T&M tab | Billable expense ($157) listed in Unbilled section; indicator: "⚡ N approved expenses not yet billed" |  |
| Tech records expense and admin approves for billing to customer | TC-11.06 Bill Expense | Click "Bill" on approved expense (with 15% markup) | Invoice line created: $157 × 1.15 = $180.55; expense status=billed; line linked to expense |  |
| Tech records expense and admin approves for billing to customer | TC-11.07 Admin Rejects (Alt) | Click "Reject" on a submitted expense | Status: submitted → rejected (red badge), visible to tech in PWA, cannot be billed |  |
| Tech records expense and admin approves for billing to customer | TC-11.08 Multiple Expenses | Add 3 expenses to 1 WO | Total expenses sum displayed at bottom; unbilled count accurate |  |

**Reference:** [StrongHeron QA Test Cases.md](../../StrongHeron%20QA%20Test%20Cases.md) — Phase 11

---

## Step 11.01 — Tech Records Expense

**Test Scenario:** Step 11.01 — Tech Records Expense

**App:** PWA → Job → Expenses (`/job/:workOrderId/expenses`)

**ExpensesScreen — Add Expense form fields:**

| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| Category | select | ✅ | From value-set expense categories |
| Vendor | input text | - | "Home Depot" |
| Description | input text | - | "Copper fittings and brazing rods" |
| Amount ($) | number | ✅ | "157.00" |
| Date | date | ✅ | Today (default) |
| Billable to customer | checkbox | - | ✅ checked |

| Test Step | Result |
|--------|----------|
| Tap "+ Add Expense" | Form opens |
| Fill fields → submit | `createExpense({ workOrderId, category, vendor, description, amountCents, expenseDate, billable })` |
| Expense created | Status = `draft` |
| Appears in expense list | Amount, vendor, date, billable badge shown |

**Expense statuses (StatusChip):**

| Status | Color | Description |
|--------|-------|-------------|
| Draft | grey (#64748b) | Newly created |
| Submitted | blue (#2563eb) | Awaiting approval |
| Approved | green (#16a34a) | Approved |
| Rejected | red (#e11d48) | Rejected |
| Reimbursed | dark green (#15803d) | Tech has been paid |

---

## Step 11.02 — Tech Submits Expense

**Test Scenario:** Step 11.02 — Tech Submits Expense

**App:** PWA → Expenses Screen

| Test Step | Result |
|--------|----------|
| Tap "Submit" on draft expense | `submitExpense(id)` called |
| Status: draft → `submitted` | Badge changes to blue |
| Expense locked from editing | Tech cannot change it |
| Appears in admin approval queue | Ready for review |

---

## Step 11.03 — Admin Views Pending Expenses

**Test Scenario:** Step 11.03 — Admin Views Pending Expenses

**App:** Admin → Expenses (`/expenses`)

**Expenses page features:**
- Filter by status: All / draft / submitted / approved / rejected / billed
- Filter by member
- Filter by date range
- Table columns: KEY, DATE, MEMBER, CATEGORY, VENDOR, DESCRIPTION, AMOUNT, MARKUP, BILLABLE, STATUS

| Test Step | Result |
|--------|----------|
| Open Expenses page | Expense list displayed |
| Filter status = "submitted" | Pending expenses only |
| Expense from tech appears | $157.00, Home Depot, billable |

---

## Step 11.04 — Admin Approves Expense

**Test Scenario:** Step 11.04 — Admin Approves Expense

**App:** Admin → Expenses

| Test Step | Result |
|--------|----------|
| Click "Approve" on expense | `approveExpense(id)` |
| Status: submitted → `approved` | Badge green |
| Cost posted to WO | WO actual cost increases by $157 |
| Expense appears in WO T&M section | Unbilled section (if billable) |

---

## Step 11.05 — Expense Appears in Unbilled

**Test Scenario:** Step 11.05 — Expense Appears in Unbilled

**App:** Admin → WO Record → T&M tab

| Test Step | Result |
|--------|----------|
| Open WO → T&M (Time & Materials) tab | Unbilled items listed |
| Billable expense ($157.00) appears in Unbilled section | Ready to bill |
| Indicator: "⚡ N approved expenses not yet billed to customer" | Count shown |

---

## Step 11.06 — Bill Expense to Customer

**Test Scenario:** Step 11.06 — Bill Expense to Customer

**App:** Admin → Expenses page or WO T&M tab

**Admin Expenses page — add expense form (more fields than PWA):**

| Field | Type | Example Data |
|-------|------|-------------|
| Category | select | Category |
| Vendor | input text | Vendor name |
| Description | input text | What was purchased |
| Amount ($) | number | Amount |
| Markup % | number | "15" (15% markup) |
| Date | date | Expense date |
| Billable | checkbox |  |

| Test Step | Result |
|--------|----------|
| Click "Bill" on approved expense | `billExpense(id)` |
| Invoice line created | Amount × (1 + markup%) |
| With 15% markup: $157 × 1.15 = $180.55 | Markup applied |
| Expense status → `billed` | Badge blue "Billed" |
| Invoice line linked to expense | Traceable |

---

## Step 11.07 — Admin Rejects Expense (Alternative Scenario)

**Test Scenario:** Step 11.07 — Admin Rejects Expense (Alternative Scenario)

**App:** Admin → Expenses

| Test Step | Result |
|--------|----------|
| Click "Reject" on submitted expense | `rejectExpense(id)` |
| Status: submitted → `rejected` | Badge red |
| Tech can see rejection in PWA | Status visible |
| Expense cannot be billed | No invoice line |

---

## Step 11.08 — Multiple Expenses + Total

**Test Scenario:** Step 11.08 — Multiple Expenses + Total

**App:** PWA → Expenses Screen

| Test Step | Result |
|--------|----------|
| Add 3 expenses to 1 WO | All created |
| Total expenses displayed at bottom | Sum of all amounts |
| "Total expenses: $XXX.XX" | Math correct |
| Unbilled count: "⚡ N approved expenses not yet billed" | Accurate count |

---

## ✅ Phase 11 Results

| Item | Status |
|------|--------|
| Tech creates expense in PWA (category, vendor, amount, billable) |  |
| Tech submits expense |  |
| Admin views pending expenses |  |
| Admin approves expense → cost posted to WO |  |
| Billable expense in Unbilled section |  |
| Bill expense → invoice line (with markup) |  |
| Admin rejects expense (alt scenario) |  |
| Multiple expenses + total calculation |  |

**Flow: Tech records → submits → admin approves → bills to customer (with markup).**
