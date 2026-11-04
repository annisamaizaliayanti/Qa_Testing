# Phase 10 — Project Multi-Phase: Phases → WOs → Milestones → Billing

> **Concept:** Large project (building renovation, multi-unit install) with several sequential phases.
> Each phase has a budget, WOs, and a milestone for billing.
> Phases can be blocked by previous phases. Change orders are available to add budget.

**Prerequisite:** Phase 1 complete. Customer "Northline School District" already exists (or create new).

---

## QA Test Case Summary

| User Story | Test Scenario | Test Step | Result | Status |
|---|---|---|---|---|
| Admin manages large project with multiple phases and budget | TC-10.01 Create Project | Fill project form: customer, name, status=active, total value=$50,000, 3 phases, start/target dates | Project created with PRJ_ key, status=active |  |
| Admin manages large project with multiple phases and budget | TC-10.02 Add Phase 1 | Add "Demolition & Prep": order=0, budget=$15,000, milestone=$15,000 | Phase created, state=pending |  |
| Admin manages large project with multiple phases and budget | TC-10.03 Add Phase 2 | Add "Installation": order=1, budget=$25,000, milestone=$25,000 | Phase created, state=pending, blocked until Phase 1 completes |  |
| Admin manages large project with multiple phases and budget | TC-10.04 Add Phase 3 | Add "Finishing & Handover": order=2, budget=$10,000, milestone=$10,000 | Phase created, state=pending; total budget = $50,000 matches project value |  |
| Admin manages large project with multiple phases and budget | TC-10.05 Create WO for Phase 1 | Create WO linked to Phase 1 with 4 line items (Demolition Labor, Crane Rental, Disposal Fee, Concrete Patching) totaling $5,530 | WO created under Phase 1, actuals tracking begins |  |
| Admin manages large project with multiple phases and budget | TC-10.05b Budget Validation (Negative) | Try adding $20,000 line item to Phase 1 WO | 422 error: "Adding this line would bring Phase 1 total to $25,530, exceeding budget of $15,000 by $10,530"; line NOT saved |  |
| Admin manages large project with multiple phases and budget | TC-10.06 Tech Completes Phase 1 Jobs | Tech completes WOs in Phase 1 | Jobs done, actual cost rises in budget tab |  |
| Admin manages large project with multiple phases and budget | TC-10.07 Add Expenses to Project | Tech adds billable expense ($157, Home Depot) via PWA, submits; admin approves | Expense posted to WO, WBS actuals updated, shows in Unbilled section |  |
| Admin manages large project with multiple phases and budget | TC-10.08 Complete Phase 1 | Click "Complete" on Phase 1 | Phase 1 state=complete, Phase 2 automatically unblocked (pending → active) |  |
| Admin manages large project with multiple phases and budget | TC-10.09 Bill Milestone Phase 1 | Click "Bill milestone" for Phase 1 | Invoice created for $15,000 linked to project, milestone status=billed |  |
| Admin manages large project with multiple phases and budget | TC-10.10 Change Order | Add change order: "Extra concrete work" +$5,000 | CO recorded, project total value rises $50,000 → $55,000 |  |
| Admin manages large project with multiple phases and budget | TC-10.11 Phase 2 Work | Create WO for Phase 2 with 4 line items ($17,490), assign tech | WO created under Phase 2, within $25,000 budget |  |
| Admin manages large project with multiple phases and budget | TC-10.12 Budget vs Actual | Open Budget tab on project record | Per-phase breakdown: budget, actual, variance displayed; change order reflected |  |
| Admin manages large project with multiple phases and budget | TC-10.13 Complete Phase 2 + Bill | Complete Phase 2, bill $25,000 milestone | Phase 2=complete, Phase 3 unblocked, invoice created |  |
| Admin manages large project with multiple phases and budget | TC-10.14 Project Done | Complete Phase 3, bill $10,000 milestone | All 3 phases complete, project status=complete, total billed = $50K + $5K CO |  |

**Reference:** [StrongHeron QA Test Cases.md](../../StrongHeron%20QA%20Test%20Cases.md) — Phase 10

---

## Step 10.01 — Create Project

**Test Scenario:** Step 10.01 — Create Project

**App:** Admin → Projects → New (`/projects/new`)

**ProjectNew form fields:**

| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| Customer | select | ✅ | "Northline School District" |
| Project name | input text | ✅ | "RTU Replacement – Northline" |
| Status | select | - | active, on_hold, complete, canceled → select **active** |
| Total Value ($) | number | - | "50000.00" |
| Total Phases | number | - | "3" |
| Notes | textarea | - | "Scope: replace all 3 rooftop units across 2 buildings" |
| Start date | date | - | Today |
| Target completion | date | - | 3 months from today |

**Duration auto-calculated:** If start + target filled, shows "X weeks" or "X days".

| Test Step | Result |
|--------|----------|
| Fill all fields → Create | Project created with `PRJ-` key |
| Redirect to project record | Project page displayed |
| Status = active | Badge green |

---

## Step 10.02 — Add Phase 1: Demolition

**Test Scenario:** Step 10.02 — Add Phase 1: Demolition

**App:** Admin → Project Record → Phases tab

**PhasesTab — Add phase form fields:**

| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| Phase name | input text | ✅ | "Phase 1 — Demolition & Prep" |
| Order | number | - | "0" (first) |
| Budget ($) | number | - | "15000.00" |
| Milestone amount ($) | number | - | "15000.00" |
| Start date | date | - | Project start date |
| End date | date | - | +2 weeks |

| Test Step | Result |
|--------|----------|
| Fill form → "Add phase" | Phase created |
| `createPhase(projectId, { name, sort, budgetCents, milestoneCents, startDate, endDate })` | API call |
| Phase state = `pending` (not started) | Initial state |

---

## Step 10.03 — Add Phase 2: Installation

**Test Scenario:** Step 10.03 — Add Phase 2: Installation

**App:** Admin → Project Record → Phases tab

| Field | Value |
|-------|-------|
| Phase name | "Phase 2 — Installation" |
| Order | "1" |
| Budget ($) | "25000.00" |
| Milestone amount ($) | "25000.00" |
| Start date | Phase 1 end date |
| End date | +4 weeks |

| Test Step | Result |
|--------|----------|
| Add phase | Phase 2 created |
| Phase 2 state = pending | Blocked until Phase 1 complete |
| Budget: $25,000 | Tracked |
| Milestone: $25,000 | Billable on completion |

---

## Step 10.04 — Add Phase 3: Finishing

**Test Scenario:** Step 10.04 — Add Phase 3: Finishing

**App:** Admin → Project Record → Phases tab

| Field | Value |
|-------|-------|
| Phase name | "Phase 3 — Finishing & Handover" |
| Order | "2" |
| Budget ($) | "10000.00" |
| Milestone amount ($) | "10000.00" |
| Start date | Phase 2 end date |
| End date | Target completion |

| Test Step | Result |
|--------|----------|
| Add phase | Phase 3 created |
| Phase 3 state = pending | Blocked until Phase 2 complete |
| Total phases: 3 | PhaseBar shows progress |
| Total budget = $50,000 ($15K + $25K + $10K) | Matches project total value |

---

## Step 10.05 — Create WO for Phase 1

**Test Scenario:** Step 10.05 — Create WO for Phase 1

**App:** Admin → Work Orders → New

**WO Form Fields:**

| Field | Value |
|-------|-------|
| Customer | "Northline School District" (auto-filled from project) |
| Project | "RTU Replacement – Northline" |
| Phase | "Phase 1 — Demolition & Prep" |
| Description | "Phase 1 Demolition & Site Prep" |

**Line Items to Add:**

| # | Description | Qty | Unit | Unit Price | Line Total |
|---|-------------|-----|------|------------|------------|
| 1 | Demolition Labor | 40 | hr | $75.00 | $3,000.00 |
| 2 | Crane / Lift Rental | 1 | day | $800.00 | $800.00 |
| 3 | Disposal Fee — Old RTU Units | 3 | ea | $350.00 | $1,050.00 |
| 4 | Concrete Patching | 8 | hr | $85.00 | $680.00 |

**Expected WO subtotal:** $5,530.00

> Budget Validation (NEW): If total of all WOs in a phase exceeds the phase budget, API returns a 422 error.
> The line items above total $5,530 — still under Phase 1 budget of $15,000 → should succeed.

| Test Step | Result |
|--------|----------|
| Create WO → link to project Phase 1 | WO created under phase |
| WO customer = project customer | Auto-matched |
| Add 4 line items above | Lines priced, subtotal = $5,530.00 |
| Schedule + assign tech | Visit ready |

**Budget Validation Test (negative case):**

| Test Step | Result |
|--------|----------|
| Try adding line item 1 × $20,000.00 to same WO | Error appears in modal |
| Error message | `Adding this line would bring "Phase 1 ..." total to $25,530.00, exceeding the phase budget of $15,000.00 by $10,530.00` |
| Line item NOT saved | WO total remains $5,530.00 |

| Expected Tracking |
|-------------------|
| WO actuals start tracking against Phase 1 budget |
| Project budget tab shows Phase 1: Budget=$15K, Actual=$5,530.00 |

---

## Step 10.05b — Test Budget Validation (Negative Case)

**Test Scenario:** Step 10.05b — Test Budget Validation (Negative Case)

**App:** Admin → Phase 1 WO (from Step 10.05) → Materials tab → Add line

> Goal: Verify that the system rejects a line item that would cause the WO total to exceed the phase budget.

| Test Step | Result |
|--------|----------|
| Open Phase 1 WO → Materials tab → "+ Add line" | Modal opens |
| Enter Description: "Over-Budget Test", Qty: 1, Rate: $20,000.00 | — |
| Click "Add line" | Error banner appears inside modal |
| Error message contains phase name, new total, budget, and difference | `Adding this line would bring "Phase 1 ..." total to $25,530.00, exceeding the phase budget of $15,000.00 by $10,530.00` |
| Modal does NOT close | Line not saved |
| WO total remains $5,530.00 | Unchanged |
| Click "Cancel" → close modal | Returns to Materials tab |

---

## Step 10.06 — Tech Completes Phase 1 Jobs

**Test Scenario:** Step 10.06 — Tech Completes Phase 1 Jobs

**App:** PWA (same as Phase 4+5)

| Test Step | Result |
|--------|----------|
| Tech completes WOs in Phase 1 | Jobs done |
| Actual cost rises | Based on WO line totals |
| Budget tab updates | Phase 1: Budget vs Actual |

---

## Step 10.07 — Add Expenses to Project

**Test Scenario:** Step 10.07 — Add Expenses to Project

**App:** PWA → Job → Expenses (`/job/:id/expenses`) or Admin

**ExpensesScreen form fields (PWA):**

| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| Category | select | ✅ | From value-set expense categories |
| Vendor | input text | - | "Home Depot" |
| Description | input text | - | "Concrete for demolition cleanup" |
| Amount ($) | number | ✅ | "157.00" |
| Date | date | ✅ | Today |
| Billable to customer | checkbox | - | ✅ checked |

**Expense statuses:**

| Status | Badge Color | Description |
|--------|-------------|-------------|
| draft | grey | Newly created |
| submitted | blue | Awaiting approval |
| approved | green | Approved |
| rejected | red | Rejected |
| reimbursed | dark green | Tech has been paid |

| Test Step | Result |
|--------|----------|
| Tech adds expense → Submit | `createExpense()` + `submitExpense()` |
| Admin approves expense | `approveExpense()` |
| Expense posted to WO | Cost tracked |
| WBS actuals increase by cost code | Budget tab updated |
| Billable expense → can be billed to customer later | Shows in Unbilled section |

---

## Step 10.08 — Complete Phase 1

**Test Scenario:** Step 10.08 — Complete Phase 1

**App:** Admin → Project Record → Phases tab

| Test Step | Result |
|--------|----------|
| Click "Complete" on Phase 1 | `completePhase(projectId, phaseId)` |
| Confirm dialog: "Mark 'Demolition' as complete?" | User confirms |
| Phase 1 state = `complete` | Badge blue "Complete" |
| **Phase 2 unblocked** | State changes from pending to active |
| Phase 2 can now be started | WOs can be created |

---

## Step 10.09 — Bill Milestone Phase 1

**Test Scenario:** Step 10.09 — Bill Milestone Phase 1

**App:** Admin → Project Record → Milestones tab

| Test Step | Result |
|--------|----------|
| "Bill milestone" for Phase 1 | `billMilestone(projectId, milestoneId)` |
| Invoice created: $15,000 | Invoice linked to project |
| Invoice customer = project customer | Correct billing |
| Milestone status → billed | Tracked |

---

## Step 10.10 — Change Order

**Test Scenario:** Step 10.10 — Change Order

**App:** Admin → Project Record → Budget/Billing section

| Test Step | Result |
|--------|----------|
| Add Change Order | CO form opens |
| Description: "Extra concrete work" | Reason documented |
| Amount: +$5,000 | Additional budget |
| Save | CO created, recorded |
| Project total value increases | $50,000 → $55,000 |
| Affected phase budget updated | If linked to specific phase |

---

## Step 10.11 — Phase 2 Begins

**Test Scenario:** Step 10.11 — Phase 2 Begins

**App:** Admin → Project Record → then Work Orders → New

**WO Form Fields:**

| Field | Value |
|-------|-------|
| Customer | "Northline School District" |
| Project | "RTU Replacement – Northline" |
| Phase | "Phase 2 — Installation" |
| Description | "Phase 2 RTU Installation" |

**Line Items to Add:**

| # | Description | Qty | Unit | Unit Price | Line Total |
|---|-------------|-----|------|------------|------------|
| 1 | RTU Unit Supply & Install | 3 | ea | $4,500.00 | $13,500.00 |
| 2 | Electrical Hookup | 3 | ea | $600.00 | $1,800.00 |
| 3 | Ductwork Modification | 16 | hr | $90.00 | $1,440.00 |
| 4 | Refrigerant Commissioning | 3 | ea | $250.00 | $750.00 |

**Expected WO subtotal:** $17,490.00

> Budget Validation: Phase 2 budget = $25,000. Total line items $17,490 → still under budget → should succeed.
> If a line item is added that brings total > $25,000, system will reject with 422 error.

| Test Step | Result |
|--------|----------|
| Phase 2 state = `active` (unblocked from Step 10.08) | Ready to work |
| Create WO for Phase 2 with 4 line items above | WO created, subtotal = $17,490.00 |
| Schedule + assign techs | Multiple visits |
| Tech executes jobs | Phase 4+5 flow |

---

## Step 10.12 — Check Budget vs Actual

**Test Scenario:** Step 10.12 — Check Budget vs Actual

**App:** Admin → Project Record → Budget tab

**Budget tracking per phase:**

| Phase | Budget | Actual | Variance |
|-------|--------|--------|----------|
| Phase 1 — Demolition | $15,000 | $14,200 | +$800 (under) |
| Phase 2 — Installation | $25,000 | $12,000 (in progress) | — |
| Phase 3 — Finishing | $10,000 | $0 (not started) | — |
| **Total** | **$55,000** | **$26,200** | — |

| Test Step | Result |
|--------|----------|
| Budget, actual, variance displayed per phase | Math correct |
| Cost codes breakdown (if applicable) | By category |
| Change order reflected | Budget adjusted |

---

## Step 10.13 — Complete Phase 2 → Bill Milestone

**Test Scenario:** Step 10.13 — Complete Phase 2 → Bill Milestone

**App:** Admin → Project Record

| Test Step | Result |
|--------|----------|
| Complete Phase 2 | State = complete, Phase 3 unblocked |
| Bill milestone Phase 2 | Invoice for $25,000 created |
| Phase 3 now active | Can create WOs |

---

## Step 10.14 — Complete Phase 3 → Bill → Project Done

**Test Scenario:** Step 10.14 — Complete Phase 3 → Bill → Project Done

**App:** Admin → Project Record

| Test Step | Result |
|--------|----------|
| Execute Phase 3 WOs | Jobs done |
| Complete Phase 3 | State = complete |
| Bill milestone Phase 3 | Invoice for $10,000 created |
| **All phases complete** | PhaseBar shows 3/3 |
| Project status → `complete` | Badge blue "Complete" |
| Total billed = total milestones | $15K + $25K + $10K = $50K (+ $5K CO) |

---

## ✅ Phase 10 Results

| Item | Status |
|------|--------|
| Project created (PRJ- key, budget, dates) |  |
| 3 phases added (sequential, budget, milestone) |  |
| Phase blocking (Phase 2 blocked by Phase 1) |  |
| WOs linked to phases, actuals tracking |  |
| Expenses linked to project WOs |  |
| Phase 1 complete → Phase 2 unblocked |  |
| Milestone billing → invoice |  |
| Change order (+$5K) |  |
| Budget vs actual tracking per phase |  |
| Sequential completion: Phase 2 → Phase 3 |  |
| All phases complete → project done |  |
| **Phase budget validation** — WO line item rejected if phase total exceeds budget |  |

**Full project lifecycle: create → phases → WOs → expenses → complete → milestone billing → change orders → budget enforcement.**
