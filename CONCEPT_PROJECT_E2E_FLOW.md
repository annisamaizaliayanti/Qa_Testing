# Concept: Project End-to-End Flow

> From the `AppMockup.html` mockup — how a project works from start to finish,
> the relationships between entities, and the workflow for each actor.

---

## 1. BIG PICTURE — What Is a "Project"?

A Project is a **container for large jobs** consisting of multiple phases, many WOs, multiple techs, and spanning weeks or months.

Difference from a regular WO:

| | Regular WO | Project |
|---|---|---|
| Duration | 1 day / 1 visit | Weeks – months |
| Scope | 1 specific job | Many jobs |
| Billing | Direct invoice | Milestone-based |
| Tracking | Simple (status) | WBS, budget vs actual |
| Example | "Fix AC unit 3" | "Replace all RTUs in buildings A + B" |

---

## QA Test Case Summary

| User Story | Test Scenario | Test Step | Result | Status |
|---|---|---|---|---|
| Admin manages a large project from WBS, milestone billing, through completion | Step 1 — Create Project | Admin → Projects → New, fill Customer, name, status, billing method, contract value, start/target dates | Project PR_xxx created (see TC-10.01) |  |
| Admin manages a large project from WBS, milestone billing, through completion | Step 2 — Create WBS (Phases + Sub-tasks) | Add Phase 1 (budget, start/end), sub-tasks 1.1/1.2, Phase 2 (blocked until Phase 1) | Phases saved, state=pending/blocked (see TC-10.02–TC-10.04) |  |
| Admin manages a large project from WBS, milestone billing, through completion | Step 3 — Set Up Milestones (billing schedule) | Add Deposit/Phase complete/Final milestones with amount per milestone | Total milestones = contract value |  |
| Admin manages a large project from WBS, milestone billing, through completion | Step 4 — Bill Deposit Milestone | Select "Deposit" milestone → Bill → send invoice | Invoice created, paid, milestone status → Paid |  |
| Admin manages a large project from WBS, milestone billing, through completion | Step 5 — Create WO per phase | Project Record → "New work order", select phase | projectId & customerId auto-filled, WO linked to phase (see TC-10.05) |  |
| Admin manages a large project from WBS, milestone billing, through completion | Step 6 — Tech executes WO | Tech: start visit, checklist, photos, materials, checkout | Phase 4+5 flow runs normally |  |
| Admin manages a large project from WBS, milestone billing, through completion | Step 7 — Tech records field expense | Enter vendor, cost code, amount, billable, markup, submit | Expense linked to WO → to Project, WBS actual increases (see TC-10.07) |  |
| Admin manages a large project from WBS, milestone billing, through completion | Step 8 — Admin monitors Budget vs Actual | Open Budget/WBS tab in Project Record | Per-phase budget, actual, variance displayed (see TC-10.12) |  |
| Admin manages a large project from WBS, milestone billing, through completion | Step 9 — Complete Phase 1 → Unblock Phase 2 | Click "Complete" on Phase 1 | Phase 1 state=complete, Phase 2 automatically unblocked (pending → active) (see TC-10.08) |  |
| Admin manages a large project from WBS, milestone billing, through completion | Step 10 — Bill Milestone Phase 1 | Select "Phase 1 complete" milestone → Bill → send invoice | Invoice created for milestone amount (see TC-10.09) |  |
| Admin manages a large project from WBS, milestone billing, through completion | Step 11 — Execute Phase 2 | Repeat Steps 5–8 for Phase 2 | Phase 2 WOs within budget, billing proceeds (see TC-10.11, TC-10.13) |  |
| Admin manages a large project from WBS, milestone billing, through completion | Step 12 — Complete Project | All phases complete, all milestones billed → Complete project | Project status=complete, total billed matches contract + change orders (see TC-10.14) |  |
| Admin manages a large project from WBS, milestone billing, through completion | Gap — WBS 2-level hierarchy (parent_id, code) | Check project_phases schema | Fields parent_id & code not present; current WBS is flat phases | ❌ Not yet |
| Admin manages a large project from WBS, milestone billing, through completion | Gap — Work Orders/Time/Files/Messages/Related/Activity tabs in Project | Open Project Record, check tab list | Those tabs are not yet available in implementation (only Expenses exists) | ❌ Not yet |

**Reference:** See related TC-IDs in [PHASE_10_PROJECT_MULTIPHASE.md](PHASE_10_PROJECT_MULTIPHASE.md), [PHASE_11_EXPENSES_FLOW.md](PHASE_11_EXPENSES_FLOW.md)

---

## 2. ENTITY RELATIONSHIP

```
                        ┌──────────────┐
                        │   CUSTOMER   │
                        │  Northline   │
                        │  School Dist │
                        └──────┬───────┘
                               │ 1
                               │
                               ▼ N
                        ┌──────────────┐
                   ┌────│   PROJECT    │────┐
                   │    │  PR_07       │    │
                   │    │  RTU program │    │
                   │    └──────┬───────┘    │
                   │           │            │
            ┌──────┘     ┌─────┴─────┐      └──────┐
            ▼            ▼           ▼             ▼
     ┌────────────┐ ┌────────┐ ┌──────────┐ ┌──────────┐
     │ MILESTONES │ │  WBS   │ │ EXPENSES │ │  FILES   │
     │ (billing   │ │(phases │ │ (project │ │(contracts│
     │  schedule) │ │ + sub- │ │  level)  │ │  plans)  │
     └────────────┘ │ tasks) │ └──────────┘ └──────────┘
                    └───┬────┘
                        │ 1 phase → N work orders
                        ▼
                 ┌──────────────┐
                 │ WORK ORDERS  │───────┬──────────┬──────────┐
                 │ WO_214, 215  │       │          │          │
                 └──────────────┘       ▼          ▼          ▼
                                  ┌──────────┐ ┌────────┐ ┌────────┐
                                  │   TIME   │ │EXPENSES│ │MATERIALS│
                                  │ ENTRIES  │ │(WO lvl)│ │(lines) │
                                  └──────────┘ └────────┘ └────────┘
```

---

## 3. WBS (Work Breakdown Structure) — Detailed Explanation

### 3.1 What is WBS?

WBS is a **hierarchical map of work** in a project. Like a document outline:

```
1      Bldg A                          ← PHASE (parent)
  1.1    RTU replacement units 1+2     ← SUB-TASK (child)
  1.2    Startup & commissioning       ← SUB-TASK (child)
2      Bldg B (blocked until 1)        ← PHASE (parent, blocked)
  2.1    Boiler tie-in                 ← SUB-TASK (child)
  2.2    Controls & closeout           ← SUB-TASK (child)
```

### 3.2 WBS Columns (from mockup)

| Column | Type | Description |
|--------|------|-------------|
| CODE | text | Hierarchy code: "1", "1.1", "1.2", "2", "2.1" |
| PHASE / TASK | text | Phase or sub-task name |
| WORK ORDERS | number | Count of WOs linked to this phase/task |
| START | date | Start date |
| END | date | End date |
| STATUS | badge | Not started / In progress / Complete |
| BUDGET | money | Allocated budget |
| ACTUAL | money | Actual cost (from WO lines + expenses + time) |
| COMPLETE | percentage | % complete |

### 3.3 WBS → Work Orders Relationship

Each WBS row (whether phase or sub-task) can have **one or more WOs** linked to it. WOs are what gets executed in the field by the tech.

```
WBS 1.1 (RTU replacement) → WO_214 (unit 1, Sam Ortiz, $4,120)
                           → WO_215 (unit 2, assigned later, $3,980)
WBS 2.1 (Boiler tie-in)   → WO_231 (blocked, $4,400)
```

### 3.4 Phase Blocking

A phase can be **blocked** by a previous phase:
- Phase 2 blocked until Phase 1 is complete
- WOs in Phase 2 show status "Blocked · phase" → cannot be worked on
- When Phase 1 is completed, Phase 2 automatically unblocks → WOs can be scheduled

### 3.5 Budget Tracking in WBS

Budget tracking happens at **every level of the WBS**:

```
Phase 1 "Bldg A"
  Budget: $7,900
  Actual: $3,374 (sum of all WOs + expenses in this phase)
  Complete: 35%

  └─ Sub-task 1.1 "RTU replacement"
       Budget: $6,400
       Actual: $3,140
       Complete: 40%
```

**Actual is calculated from:**
- WO line totals (material + labor)
- Time entries (hours × rate)
- Expenses (approved, linked to WOs in this phase)

---

## 4. END-TO-END WORKFLOW

### STEP 1: Admin Creates Project

**Who:** Admin/Project Manager
**Where:** Admin → Projects → + New project

**Form fields:**
| Field | Value |
|-------|-------|
| Customer | "Northline School District" (select) |
| Project name | "RTU program · Bldg A + B" |
| Status | Active |
| Billing method | Milestone |
| Contract value | $14,800.00 |
| Start | 09/08/2026 |
| Target end | 10/17/2026 |

**Result:** Project `PR_07` created. Redirects to Project Record.

---

### STEP 2: Admin Creates WBS (Phases + Sub-tasks)

**Who:** Admin/Project Manager
**Where:** Admin → Project Record → WBS tab

**Phase 1:**
| Field | Value |
|-------|-------|
| Code | 1 |
| Phase name | "Bldg A" |
| Budget | $7,900.00 |
| Start | 09/08/2026 |
| End | 09/26/2026 |

**Sub-task 1.1:**
| Field | Value |
|-------|-------|
| Code | 1.1 |
| Task name | "RTU replacement · units 1 + 2" |
| Parent | Phase 1 |
| Budget | $6,400.00 |
| Start | 09/08/2026 |
| End | 09/19/2026 |

**Sub-task 1.2:**
| Field | Value |
|-------|-------|
| Code | 1.2 |
| Task name | "Startup & commissioning" |
| Parent | Phase 1 |
| Budget | $1,500.00 |
| Start | 09/22/2026 |
| End | 09/26/2026 |

**Phase 2 (blocked until Phase 1):**
| Field | Value |
|-------|-------|
| Code | 2 |
| Phase name | "Bldg B" |
| Budget | $5,300.00 |
| Start | 09/29/2026 |
| End | 10/17/2026 |
| Blocked until | Phase 1 |

**Sub-tasks 2.1 and 2.2 created similarly.**

**Result:** WBS table displays with hierarchy codes, budget per row, status "Not started".

---

### STEP 3: Admin Sets Up Milestones (Billing Schedule)

**Who:** Admin
**Where:** Admin → Project Record → Billing tab → MILESTONES section

Milestones determine **when and how much to bill** the customer:

| Milestone | Amount | Trigger |
|-----------|--------|---------|
| Deposit | $4,440.00 (30%) | Contract signed |
| Phase 1 complete | $5,920.00 (40%) | Phase 1 done |
| Final | $4,440.00 (30%) | Project complete |

**Total milestones = $14,800 = contract value** ← must match

---

### STEP 4: Admin Bills Deposit Milestone

**Who:** Admin
**Where:** Project Record → Actions → "Bill milestone…"

| Action | Result |
|--------|--------|
| Select "Deposit" milestone | Invoice `INV-1043` created, $4,440.00 |
| Send invoice to customer | Customer pays |
| Invoice paid | Milestone status → "Paid" |

**Project Summary now:**
```
Billed to date:   $4,440.00
Balance to bill:  $10,360.00
```

---

### STEP 5: Admin Creates Work Orders for Phase 1

**Who:** Admin
**Where:** Project Record → "New work order" button

When clicking "New work order" from Project Record:
- `projectId` is automatically set to this project
- `customerId` is automatically the same as the project customer
- Admin selects **which phase** this WO belongs to

**WO 1 — RTU unit 1:**
| Field | Value |
|-------|-------|
| Project | PR_07 (auto-filled) |
| Phase/WBS item | 1.1 — RTU replacement |
| Customer | Northline School District (auto-filled) |
| Site | Bldg A · 1400 Northline Rd |
| Job type | RTU install |
| Assigned tech | Sam Ortiz |
| Lines | RTU replacement 5 ton ($3,740) + Startup ($380) |

**WO 2 — RTU unit 2:**
| Field | Value |
|-------|-------|
| Project | PR_07 (auto-filled) |
| Phase/WBS item | 1.1 — RTU replacement |
| Lines | Similar items, $3,980 |

**Result:**
- 2 WOs created, linked to project PR_07, phase 1.1
- WBS tab updated: "1.1 RTU replacement — WORK ORDERS: 2"
- Work Orders tab in Project shows WO_214 and WO_215

**Relationship in WO Record:**
In WO_214 → Overview tab → RELATED section:
```
Quote       Q_118
Project     PR_07
Equipment   RTU-2 · Bldg A roof
```

---

### STEP 6: Tech Executes WO (Phase 4+5 flow)

**Who:** Tech (Sam Ortiz)
**Where:** PWA → My Day → Job

Same flow as Phase 4+5 in the test plan:
1. Tech views job in My Day
2. Start visit → timer starts
3. Complete checklist
4. Take photos (before/after)
5. Record materials (items used in the field)
6. Stop timer → time entry created
7. Checkout → customer sign → payment

**Project-related tracking:**
- Time entries automatically link to WO_214 → link to Project PR_07
- Materials/expenses in WO → tracked against project budget
- Photos → visible at project level

---

### STEP 7: Tech Records Field Expense

**Who:** Tech
**Where:** PWA → Job → Materials (or Expenses)

**Example:** Tech buys additional material not on the WO:
| Field | Value |
|-------|-------|
| Vendor | Ferguson |
| Cost code | MT-300 (Materials) |
| Amount | $1,760.00 |
| Billable | Yes |
| Markup | 10% |

**This expense:**
- Linked to WO_214 → linked to Project PR_07
- Appears in Project → Expenses tab
- Appears in Project → WBS → Phase 1 → Actual increases by $1,760

---

### STEP 8: Admin Monitors in Project Record

**Who:** Admin/PM
**Where:** Project Record

**Overview tab → Summary:**
```
Contract value:   $14,800.00
Billed to date:   $4,440.00
Balance to bill:  $10,360.00
Cost to date:     $3,374.00
Committed:        $1,180.00
```

**WBS tab:**
```
CODE  PHASE/TASK              WOs  START  END    STATUS       BUDGET    ACTUAL    %
1     Bldg A                  2    09/08  09/26  In progress  $7,900    $3,374    35%
1.1   RTU replacement u1+2    2    09/08  09/19  In progress  $6,400    $3,140    40%
1.2   Startup & commissioning —    09/22  09/26  Not started  $1,500    $234      0%
2     Bldg B (blocked)        1    09/29  10/17  Not started  $5,300    $0        0%
```

**Work Orders tab:**
```
WO      PHASE  TYPE         STATUS     SCHEDULED   TOTAL
WO_214  1      RTU install  Scheduled  09/08/2026  $4,120.00
WO_215  1      RTU install  New        —           $3,980.00
WO_231  2      Boiler tie   Blocked    —           $4,400.00
```

**Time tab:**
```
DATE   MEMBER       WO      TYPE        CODE    DURATION  BILLABLE  STATUS
09/08  Sam Ortiz    WO_214  Site labor  LB-200  2.5       Yes       Submitted
09/08  Priya Shah   WO_214  Site labor  LB-200  2.5       Yes       Submitted
09/05  Sam Ortiz    WO_214  Shop        SH-100  1.0       —         Approved
```

**Expenses tab:**
```
DATE   VENDOR           CODE    WO      AMOUNT    BILLABLE  MARKUP  STATUS
09/09  United Rentals   EQ-400  WO_214  $450.00   Yes       15%     Approved
09/06  Ferguson         MT-300  WO_214  $1,760    Yes       10%     Approved
09/04  Parking · county OF-900  —       $18.00    —         —       Approved
```

**Billing tab:**
```
MILESTONES
Deposit             $4,440   Contract signed   INV-1043   Paid
Phase 1 complete    $5,920   Phase 1 done      —          Not billed
Final               $4,440   Project complete  —          Not billed

BUDGET SUMMARY
Contract:           $14,800
Billed to date:     $4,440
Balance to bill:    $10,360
Labor cost to date: $1,164
Materials cost:     $2,210
```

**Files tab:**
```
NAME                      TYPE  SIZE    BY         ADDED
Signed contract.pdf       PDF   240 KB  System     09/02/2026
Roof plan · Bldg A.pdf    PDF   3.4 MB  A. Carter  09/03/2026
```

---

### STEP 9: Complete Phase 1 → Unblock Phase 2

**Who:** Admin
**Where:** Project Record → Actions → or WBS tab

| Action | Result |
|--------|--------|
| All WOs in Phase 1 complete | Phase 1 can be completed |
| Click "Complete Phase 1" | Phase 1 status → Complete |
| Phase 2 auto-unblock | Status → Active, WO_231 can be scheduled |
| Header badge changes | "Phase 2 of 2" |

---

### STEP 10: Bill Milestone Phase 1

**Who:** Admin
**Where:** Project Record → Actions → "Bill milestone…"

| Action | Result |
|--------|--------|
| Select "Phase 1 complete" milestone | Invoice created for $5,920 |
| Send invoice | Customer pays |

**Summary now:**
```
Billed to date:   $10,360.00  ($4,440 + $5,920)
Balance to bill:  $4,440.00
```

---

### STEP 11: Execute Phase 2

Repeat Steps 5-8 for Phase 2:
- Create new WOs for Phase 2
- Tech executes
- Record time, expenses, materials
- Budget vs actual tracking

---

### STEP 12: Complete Project

**Who:** Admin
**Where:** Project Record → Actions → "Complete project"

| Action | Result |
|--------|--------|
| All phases complete | All WBS items green |
| All milestones billed | All invoices paid |
| Complete project | Status → Complete |

**Final Summary:**
```
Contract value:    $14,800.00
Billed to date:    $14,800.00
Balance to bill:   $0.00
Cost to date:      $12,500.00
Profit:            $2,300.00
```

---

## 5. ENTITY RELATIONSHIPS — DETAIL

### 5.1 Project → WBS (Phases + Sub-tasks)

```
projects (1) ──→ (N) project_phases
                      │
                      ├── id
                      ├── project_id (FK → projects)
                      ├── parent_id (FK → project_phases, NULL = root phase)
                      ├── code ("1", "1.1", "1.2")
                      ├── name
                      ├── sort
                      ├── status (pending / active / in_progress / complete)
                      ├── block_until_phase_id (FK → project_phases)
                      ├── budget_cents
                      ├── actual_cached_cents (computed from WO + expenses)
                      ├── start_date
                      └── end_date
```

**Not yet in current schema:**
- `parent_id` (for sub-task hierarchy)
- `code` (for WBS codes like "1.1")

### 5.2 WBS → Work Orders

```
project_phases (1) ──→ (N) work_orders
                             │
                             ├── project_id (FK → projects)
                             ├── phase_id (FK → project_phases) ← NOT YET
                             └── status: if phase blocked → WO "Blocked · phase"
```

### 5.3 Work Orders → Time/Expenses/Materials

```
work_orders (1) ──→ (N) time_entries    (via work_order_id)
              (1) ──→ (N) expenses       (via work_order_id)
              (1) ──→ (N) wo_lines       (materials/items)
```

All of these **roll up** to project level:
- Project → Time tab = all time_entries from all WOs in this project
- Project → Expenses tab = expenses directly on project + expenses from WOs in project
- Project → WBS actual = sum(WO totals + expenses) per phase

### 5.4 Project → Milestones → Invoices

```
projects (1) ──→ (N) project_milestones
                       │
                       ├── name ("Deposit", "Phase 1 complete")
                       ├── amount_cents
                       ├── trigger (when billing is allowed)
                       ├── phase_id (optional, linked to phase)
                       ├── invoiced (boolean)
                       └── invoice_id (FK → invoices, after billed)
```

### 5.5 Project → Files

```
files
  │
  ├── object = "project"
  ├── record_id = project.id
  ├── kind = "pdf" / "doc" / "photo"
  └── original_name = "Signed contract.pdf"
```

---

## 6. GAP: CURRENT IMPLEMENTATION vs MOCKUP

### 6.1 Database Schema — Fields NOT YET Present

| Table | New Field | Type | Function |
|-------|-----------|------|----------|
| `projects` | `billing_method` | varchar | "milestone" / "T&M" / "fixed" |
| `projects` | `project_manager_id` | text FK→members | PM assignment |
| `projects` | `billed_cents` | integer | Cached total billed |
| `projects` | `committed_cents` | integer | Cached committed cost |
| `project_phases` | `parent_id` | text FK→project_phases | Sub-task hierarchy |
| `project_phases` | `code` | varchar | WBS code ("1", "1.1") |
| `project_phases` | `actual_cached_cents` | integer | Cached actual cost |
| `work_orders` | `phase_id` | text FK→project_phases | Link WO to phase |
| `project_milestones` | `trigger` | varchar | Description of when to bill |

### 6.2 API Endpoints — NOT YET Present

| Method | Endpoint | Function |
|--------|----------|----------|
| GET | `/projects/:id/work-orders` | List WOs by project |
| GET | `/projects/:id/time-entries` | List time entries via WO join |
| GET | `/projects/:id/files` | List files by project |
| POST | `/projects/:id/files` | Upload file to project |
| GET | `/projects/:id/activity` | Project audit log |
| GET | `/projects/:id/messages` | Messages linked to project |
| POST | `/work-orders` | Add support for `projectId`, `phaseId` |

### 6.3 Frontend Pages — NOT YET Present

| Component | Status | Notes |
|-----------|--------|-------|
| "Overview" tab (rename "Details") | Partial | Needs SUMMARY section added |
| "WBS" tab (upgrade "Phases") | ❌ | Needs upgrade to hierarchy table |
| "Work Orders" tab | ❌ | Not yet present |
| "Time" tab | ❌ | Not yet present |
| "Expenses" tab | ✅ | Already present |
| "Billing" tab (merge Milestones + Budget) | Partial | Milestones present, budget summary missing |
| "Files" tab | ❌ | Not yet present |
| "Messages" tab | ❌ | Not yet present |
| "Related" tab | ❌ | Not yet present |
| "Activity" tab | ❌ | Not yet present |
| Header "New work order" button | ❌ | Not yet present |
| Header "Actions" dropdown | ❌ | Not yet present |
| WO New form — Project/Phase field | ❌ | Not yet present |

### 6.4 Projects List — Gap

**Mockup columns:** PR_#, PROJECT, CUSTOMER, PHASE, WORK ORDERS, BILLED, BALANCE

**Implementation columns:** Project #, Name/Customer, Status, Phases (bar), Start, Target, Value

| Column | Mockup | Implementation | Gap |
|--------|--------|----------------|-----|
| PHASE | "1 of 2 · Bldg A" (active phase name) | PhaseBar (progress bar) | Different format |
| WORK ORDERS | Count ("2") | ❌ Not present | Needs to be added |
| BILLED | "$1,236.00" | ❌ Not present | Needs to be added |
| BALANCE | "$2,884.00" | ❌ Not present | Needs to be added |
| Status | Not in mockup | ✅ Present | Mockup does not use status badge |

---

## 7. END-TO-END FLOW SUMMARY

```
┌─────────────────────────────────────────────────────────────────┐
│                     ADMIN / PROJECT MANAGER                      │
│                                                                  │
│  1. Create Project (customer, contract value, dates)             │
│  2. Create WBS (phases + sub-tasks, budget per row)              │
│  3. Set up Milestones (billing schedule)                         │
│  4. Bill deposit milestone → invoice → customer pays             │
│  5. Create WOs per phase (linked to WBS item)                    │
│  6. Schedule + assign tech to WO                                 │
│                        │                                         │
│                        ▼                                         │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │                    TECH IN THE FIELD                     │     │
│  │                                                          │     │
│  │  7. Execute WO (checklist, photos)                       │     │
│  │  8. Record time (timer / manual entry)                   │     │
│  │  9. Record materials / expenses                          │     │
│  │  10. Checkout + payment (if there is a balance on WO)    │     │
│  └─────────────────────────────────────────────────────────┘     │
│                        │                                         │
│                        ▼                                         │
│  11. Monitor: WBS actual vs budget, time, expenses               │
│  12. Approve expenses + time entries                             │
│  13. Complete Phase 1 → Phase 2 unblocks                         │
│  14. Bill milestone Phase 1 → invoice                            │
│  15. Repeat for Phase 2, 3, ...                                  │
│  16. All phases complete → bill final milestone                  │
│  17. Complete project                                            │
│                                                                  │
│  FINAL STATE:                                                    │
│  ✅ All WBS items complete                                       │
│  ✅ All milestones billed + paid                                 │
│  ✅ Budget vs actual tracked                                     │
│  ✅ All files documented                                         │
│  ✅ Project status = Complete                                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. QUESTIONS FOR CLIENT

Before implementation, the following need confirmation:

1. **WBS hierarchy** — Is 2-level hierarchy (phase → sub-task) as in the mockup needed, or is flat phases as currently implemented sufficient?

2. **Phase blocking** — Can sub-tasks within a phase also block each other, or is blocking only between top-level phases?

3. **Billing method** — Is milestone-only billing needed, or also T&M (Time & Materials) and fixed price?

4. **Project manager field** — Is the PM always one of the members, or can it be someone outside the system?

5. **Cost code** — Is cost code tracking (LB-200, MT-300, EQ-400) required on time entries and expenses? A `cost_codes` table already exists in the DB.

6. **WO creation** — Can WOs **only** be created from within the Project Record, or can they also be created from the regular WO list and linked to a project afterward?

7. **Materials vs Expenses** — In the tech PWA, should "Materials" (item lines from WO) remain alongside "Expenses" (ad-hoc field purchases), or just one of them?
