# Phase 12 — Time Entry Flow: Tech Timer → Submit Week → Admin Approve → Payroll

> **Concept:** Tech records work hours during the week (timer + manual entry), submits the week, admin reviews flags (overtime, overlap), approves, then exports a payroll batch.
> Billable time can be billed to customer via invoice.

**Prerequisite:** An active WO assigned to tech exists. Phase 4 flow (timer) already familiar.

---

## QA Test Case Summary

| User Story | Test Scenario | Test Step | Result | Status |
|---|---|---|---|---|
| Tech records work hours and admin approves for payroll | TC-12.01 Timer Start | Open job → Time tile, select entry type, tap "Start timer" | Timer running, state saved in localStorage sh_timer_{visitId}, elapsed seconds counting |  |
| Tech records work hours and admin approves for payroll | TC-12.02 Timer Stop → Entry | Tap "Stop timer" | Timer stops, localStorage cleared, start rounded down 15 min, end rounded up 15 min (min 15-min session), time entry auto-created with source=timer |  |
| Tech records work hours and admin approves for payroll | TC-12.03 Manual Entry | Tap "Add entry", select type="Travel", set start/end (45 min), add memo | Entry created with type, duration, billable flag, memo |  |
| Tech records work hours and admin approves for payroll | TC-12.04 Multi-WO Entries | Open different job, add time entry | Entry linked to second WO; multiple entries tracked per WO |  |
| Tech records work hours and admin approves for payroll | TC-12.05 Submit Week | Submit all entries for the week | Entries status → submitted, locked from editing, appear in admin approval queue |  |
| Tech records work hours and admin approves for payroll | TC-12.06 Admin Week Grid | Open /time, select member | Week grid shows daily totals (billable/non-billable/total hours), stat tiles (total hours, billable %, status badge) |  |
| Tech records work hours and admin approves for payroll | TC-12.07 Admin Flags | Check Approvals tab for flags | Flags detected: >13h day ("⚠ 13h day flagged"), overlapping entries, weekend work |  |
| Tech records work hours and admin approves for payroll | TC-12.08 Admin Approve | Click "Approve" (with acknowledgeFlags if needed) | Entries status → approved (green badge), edit blocked, toast confirmation |  |
| Tech records work hours and admin approves for payroll | TC-12.09 Admin Adjust Entry | Select one entry, adjust duration (-1 hour) | Adjustment row created, original preserved (audit trail), net hours = original - adjustment |  |
| Tech records work hours and admin approves for payroll | TC-12.10 Billable Time → Invoice | Open WO → T&M tab, click "Add to invoice" on billable entries | Labor line created on invoice: hours × mapped rate = line total, linked to time entries |  |
| Tech records work hours and admin approves for payroll | TC-12.11 Payroll Export | Click "Export" payroll, select period | Payroll batch created with PR_ key, entries status → locked (immutable), export result: key, id, entryCount, delivery |  |

**Reference:** [StrongHeron QA Test Cases.md](../../StrongHeron%20QA%20Test%20Cases.md) — Phase 12

---

## Step 12.01 — Tech Starts Timer on Job

**Test Scenario:** Step 12.01 — Tech Starts Timer on Job

**App:** PWA → Job → Time (`/job/:workOrderId/time`)

| Test Step | Result |
|--------|----------|
| Open job → tap Time tile | Time screen opens |
| Select entry type from dropdown | Value-set `entry_type` (cached in localStorage `sh_entry_types`) |
| Tap "Start timer" | Timer starts running |
| Timer state saved in `localStorage` key `sh_timer_{visitId}` | Survives page refresh |
| Elapsed time counting (seconds) | UI updates every second |

---

## Step 12.02 — Tech Stops Timer → Entry Created

**Test Scenario:** Step 12.02 — Tech Stops Timer → Entry Created

**App:** PWA → Time Screen

| Test Step | Result |
|--------|----------|
| Tap "Stop timer" | Timer stops |
| Start time rounded down 15 min | `roundDown15(timerStart)` |
| End time rounded up 15 min | Min 15-min session enforced |
| Time entry auto-created | `createTimeEntry(params)` |
| Entry source = `timer` | Distinguishable from manual |

**Auto-generated entry params:**

| Field | Value |
|-------|-------|
| entryTypeValueId | Selected entry type |
| workOrderId | Current WO |
| date | Timer start date (YYYY-MM-DD) |
| startTime | Rounded start (HH:MM) |
| endTime | Rounded end (HH:MM) |

| Test Step | Result |
|--------|----------|
| Entry appears in list | Date, start, end, duration, type visible |
| `localStorage` key cleared | Timer state removed |

---

## Step 12.03 — Tech Manual Entry

**Test Scenario:** Step 12.03 — Tech Manual Entry

**App:** PWA → Time Screen

| Test Step | Result |
|--------|----------|
| Tap "Add entry" (manual) | Form opens |
| Entry type = "Travel" | Select from dropdown |
| Duration = 45 min | Start/end time or duration |
| Memo = "Drive to second site" | Note field |
| Save | Entry created |

**Manual entry fields:**

| Field | Type | Example |
|-------|------|--------|
| Entry type | select | Travel, Service, Admin, etc. |
| Date | date | Today |
| Start time | time | "14:00" |
| End time | time | "14:45" |
| Billable | checkbox | ✅ or ☐ |
| Cost code | select | Optional |
| Memo | text | "Drive to second site" |

---

## Step 12.04 — Tech Entry on Another Job

**Test Scenario:** Step 12.04 — Tech Entry on Another Job

**App:** PWA → different job → Time

| Test Step | Result |
|--------|----------|
| Open another job → Time tile | Time screen for different WO |
| Start timer / add manual entry | Entry linked to this WO |
| Multiple entries across different WOs | All tracked per WO |

---

## Step 12.05 — Tech Submits Week

**Test Scenario:** Step 12.05 — Tech Submits Week

**App:** PWA or Admin

| Test Step | Result |
|--------|----------|
| Tech submits all entries for the week | `submitTimesheet(memberId, week)` |
| Entries status → `submitted` | Locked from editing |
| Appears in admin approval queue | Ready for review |

---

## Step 12.06 — Admin Views Week Grid

**Test Scenario:** Step 12.06 — Admin Views Week Grid

**App:** Admin → Time (`/time`)

**Time page views:**
- **Week grid** per member: rows = days, columns = hours
- **Entry list**: individual entries with details
- **Approvals**: submitted timesheets pending review

**Week grid columns:**

| Column | Description |
|--------|-------------|
| DATE | Day of week |
| ENTRIES | Count of entries |
| BILLABLE | Billable hours |
| NON-BILLABLE | Non-billable hours |
| TOTAL | Total hours |
| STATUS | draft/submitted/approved/locked |

**Entry list columns:**

| Column | Description |
|--------|-------------|
| DATE | Entry date |
| MEMBER | Tech name |
| TYPE | Entry type (Service, Travel, etc.) |
| WORK ORDER | WO key link |
| DURATION | Hours:minutes |
| BILLABLE | Yes/No |
| NOTE | Memo text |
| STATUS | Badge |

**Stat tiles:**

| Tile | Description |
|------|-------------|
| Total hours | Sum of all entries in period |
| Billable | Billable hours + percentage |
| Status | Current timesheet status badge |

**Filters:**

| Filter | Type | Options |
|--------|------|---------|
| MEMBER | select | Me / all team members |
| STATUS | select | All / draft / submitted / approved / locked |
| Week picker | select | Week labels (Mon-Sun ranges) |

| Test Step | Result |
|--------|----------|
| Select member who submitted | Entries shown |
| Week grid shows daily totals | Hours per day |
| Status = submitted | Blue badge |

---

## Step 12.07 — Admin Reviews Flags

**Test Scenario:** Step 12.07 — Admin Reviews Flags

**App:** Admin → Time → Approvals view

**Flag types:**

| Flag | Trigger | Description |
|------|---------|-------------|
| Long day | >13 hours in one day | "⚠ 13h day flagged" |
| Overlap | Two entries overlap times | "⚠ Overlapping entries" |
| Weekend | Entries on Sat/Sun | "⚠ Weekend work" |

| Test Step | Result |
|--------|----------|
| Approvals tab → check flags | "⚠ N flags" indicator |
| Flag details visible | Specific issues listed |
| Admin can acknowledge flags and still approve | `acknowledgeFlags: true` |

---

## Step 12.08 — Admin Approves Week

**Test Scenario:** Step 12.08 — Admin Approves Week

**App:** Admin → Time → Approvals

**ApprovalsScreen (PWA) tabs:**
- **Time** tab: `listTimeApprovals()` → pending timesheets
- **Payments** tab: `listPaymentApprovals('pending')` → pending payment requests

| Test Step | Result |
|--------|----------|
| Click "Approve" on timesheet | `approveTimesheet(memberId, week, { acknowledgeFlags })` |
| Entries status → `approved` | Green badge |
| Entries edit blocked | Cannot modify |
| Toast: "Timesheet approved" | Confirmation |

**Return timesheet (alt):**

| Field | Type | Required |
|-------|------|----------|
| Note to member | textarea |  |

| Test Step | Result |
|--------|----------|
| Click "Return" → add note → submit | `returnTimesheet(memberId, week, { note })` |
| Entries back to draft | Tech can edit and resubmit |

---

## Step 12.09 — Admin Adjusts Entry

**Test Scenario:** Step 12.09 — Admin Adjusts Entry

**App:** Admin → Time → Entry detail

| Test Step | Result |
|--------|----------|
| Select one entry | Entry detail view |
| Adjust duration (reduce by 1 hour) | Adjustment UI |
| Save adjustment | New adjustment row created |
| Original entry preserved | Audit trail |
| Net hours = original - adjustment | Math correct |

---

## Step 12.10 — Billable Time → Invoice

**Test Scenario:** Step 12.10 — Billable Time → Invoice

**App:** Admin → WO Record → T&M tab

| Test Step | Result |
|--------|----------|
| Open WO → T&M (Time & Materials) | Billable time entries listed |
| Click "Add to invoice" | Labor line created on invoice |
| Rate = mapped item rate (from labor type → item mapping) | Correct billing rate |
| Hours × rate = line total | Math correct |
| Invoice line linked to time entries | Traceable |

---

## Step 12.11 — Export Payroll Batch

**Test Scenario:** Step 12.11 — Export Payroll Batch

**App:** Admin → Time

| Test Step | Result |
|--------|----------|
| Click "Export" payroll | Export flow initiated |
| Select period / approved entries | Batch scoped |
| Generate payroll batch | `PR_` key prefix |
| Entries status → `locked` | Immutable after export |
| Export result: `{ key, id, entryCount, delivery }` | Batch metadata |
| Entries cannot be edited or un-approved | Permanently locked |

---

## ✅ Phase 12 Results

| Item | Status |
|------|--------|
| Tech starts timer → auto time entry |  |
| Tech manual time entry (type, duration, memo) |  |
| Multiple entries across different WOs |  |
| Submit week → locked |  |
| Admin week grid view (daily totals) |  |
| Flags detection (13h day, overlap, weekend) |  |
| Admin approves timesheet (acknowledge flags) |  |
| Admin returns timesheet (with note) |  |
| Adjust entry (adjustment row, audit trail) |  |
| Billable time → invoice line |  |
| Payroll export → PR_ batch → entries locked |  |

**Full time lifecycle: timer → entries → submit → flags → approve → bill/payroll export.**
