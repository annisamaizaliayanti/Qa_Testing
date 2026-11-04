# Phase 13 — Owner Mobile Flow: Monitor + Assign + Approve

> **Concept:** Owner/dispatcher uses the PWA mobile to monitor operations, assign tech to jobs, approve expenses/time, and communicate — all from a mobile device.

**Prerequisite:** Phase 1 complete. Several active WOs exist, time entries submitted, expenses pending.

---

## QA Test Case Summary

| User Story | Test Scenario | Test Step | Result | Status |
|---|---|---|---|---|
| Owner/dispatcher monitors operations via PWA mobile | TC-13.01 Owner Login | Login as admin/dispatcher in PWA | Redirect to /today showing OwnerTodayScreen (not tech TodayScreen) |  |
| Owner/dispatcher monitors operations via PWA mobile | TC-13.02 Today Overview | View dashboard | Stat tiles: today's visits, open WOs, approvals count, week stats; visit list with customer, time, tech (or "⚠ Unassigned" warning) |  |
| Owner/dispatcher monitors operations via PWA mobile | TC-13.03 Assign Tech | Open unassigned job, tap assignment, select tech | Assignment updated, "Unassigned" warning gone, tech receives notification |  |
| Owner/dispatcher monitors operations via PWA mobile | TC-13.04 Schedule View | Navigate to /schedule | Week view with day-by-day visits, per-member rows, week navigation |  |
| Owner/dispatcher monitors operations via PWA mobile | TC-13.05 Approve Timesheet | Open /approvals → Time tab, tap "Approve" on submitted timesheet | Timesheet approved, toast confirmation; or "Return" with note sends back to tech |  |
| Owner/dispatcher monitors operations via PWA mobile | TC-13.06 Approve/Reject Payment | Open /approvals → Payments tab | Pending payment requests listed; approve or reject with actions |  |
| Owner/dispatcher monitors operations via PWA mobile | TC-13.07 Messages | Navigate to /messages, tap thread | Conversation history visible, type reply and send; customer receives SMS/email |  |
| Owner/dispatcher monitors operations via PWA mobile | TC-13.08 Notifications | Navigate to /notifications | Notification list with job updates, approvals, messages, alerts; tap navigates to relevant screen; mark as read |  |

**Reference:** [StrongHeron QA Test Cases.md](../../StrongHeron%20QA%20Test%20Cases.md) — Phase 13

---

## Step 13.01 — Owner Logs into PWA

**Test Scenario:** Step 13.01 — Owner Logs into PWA

**App:** PWA (localhost:5175)

| Test Step | Result |
|--------|----------|
| Open `/login` | Login screen displayed |
| Login as admin/dispatcher (not tech) | Auth succeeds |
| Redirect to `/today` | **OwnerTodayScreen** shown (not TodayScreen for tech) |

**OwnerTodayScreen vs TodayScreen:** Role-based — admin/dispatcher sees overview, tech sees personal schedule.

---

## Step 13.02 — View Today Overview

**Test Scenario:** Step 13.02 — View Today Overview

**App:** PWA → `/today` (OwnerTodayScreen)

**Stat tiles:**

| Tile | Data Source | Example |
|------|------------|--------|
| Today's visits | `summary.todayVisitCount` | "5" |
| Open WOs | `summary.openWorkOrders` | "12" |
| Approvals | Pending count | "3" |
| This week | Week stats | "18 visits" |

**Visit list (today):**

| Info | Description |
|------|-------------|
| Customer name | Who the visit is for |
| Time slot | Scheduled time |
| Tech assigned | Name, or "⚠ Unassigned" (amber warning) |
| Status | Visit status badge |

| Test Step | Result |
|--------|----------|
| Dashboard overview displayed | Stats + visit list |
| Unassigned visits highlighted | "⚠ N unassigned" warning count |
| Tap visit → navigate to job | Job detail accessible |

---

## Step 13.03 — Assign Tech to Job

**Test Scenario:** Step 13.03 — Assign Tech to Job

**App:** PWA → Job Detail

| Test Step | Result |
|--------|----------|
| Open unassigned job | Job screen displayed |
| Tap "Assigned" / change assignment | Tech picker opens |
| Select tech from dropdown | Assignment updated |
| Visit card updated | Tech name shown, "Unassigned" warning gone |
| Tech receives notification | Job appears in their My Day |

---

## Step 13.04 — View Schedule

**Test Scenario:** Step 13.04 — View Schedule

**App:** PWA → `/schedule`

| Test Step | Result |
|--------|----------|
| Tap "Schedule" from OwnerTodayScreen | Schedule screen opens |
| This week's visits shown | Day-by-day view |
| All tech schedules visible | Per-member rows |
| Navigate between weeks | Week picker |

---

## Step 13.05 — Approve from Mobile

**Test Scenario:** Step 13.05 — Approve from Mobile

**App:** PWA → `/approvals`

**ApprovalsScreen tabs:**

### Time tab

| Column | Description |
|--------|-------------|
| Member name | Who submitted |
| Week | Date range |
| Hours | Total hours |
| Flags | Count if any |
| Actions | Approve / Return |

### Payments tab

| Column | Description |
|--------|-------------|
| Description | Payment request detail |
| Amount | Requested amount |
| Actions | Approve / Reject |

| Test Step | Result |
|--------|----------|
| "Time" tab → view pending timesheets | Submitted timesheets listed |
| Tap "Approve" | `approveTimesheet(memberId, week, { acknowledgeFlags })` |
| Toast: "Timesheet approved" | Confirmation |
| "Payments" tab → view pending requests | Payment approvals listed |
| Approve / reject payment request | `approvePayment(id)` / `rejectPayment(id)` |

**Return timesheet flow:**

| Field | Type | Required |
|-------|------|----------|
| Note to member | textarea |  |

| Test Step | Result |
|--------|----------|
| "Return" → add reason → submit | Timesheet returned to tech |

---

## Step 13.06 — View Messages

**Test Scenario:** Step 13.06 — View Messages

**App:** PWA → `/messages`

| Test Step | Result |
|--------|----------|
| Tap "Messages" | Thread list shown |
| Conversations with customers | Sorted by most recent |
| Unread indicators | Badge counts |

---

## Step 13.07 — Reply to Message

**Test Scenario:** Step 13.07 — Reply to Message

**App:** PWA → `/messages` → Thread (`/thread/:id`)

| Test Step | Result |
|--------|----------|
| Tap conversation thread | Thread screen opens |
| Message history displayed | Full conversation |
| Type reply → Send | Message sent to customer |
| Customer receives message | SMS/email delivered |

---

## Step 13.08 — Notifications

**Test Scenario:** Step 13.08 — Notifications

**App:** PWA → `/notifications`

| Test Step | Result |
|--------|----------|
| Tap notification icon / navigate to notifications | Notification list displayed |
| Different notification types | Job updates, approvals, messages, alerts |
| Tap notification → navigate to relevant screen | Deep link works |
| Mark as read | Visual update |

---

## ✅ Phase 13 Results

| Item | Status |
|------|--------|
| Owner login → OwnerTodayScreen (not tech view) |  |
| Today overview: stats, visits, unassigned warnings |  |
| Assign tech to job from mobile |  |
| Schedule view (week, per-member) |  |
| Approve timesheets from mobile |  |
| Approve/reject payment requests |  |
| Return timesheet with note |  |
| Messages — view threads + reply |  |
| Notifications — list + navigate |  |

**Owner can manage entire business from mobile: monitor, assign, approve, communicate.**
