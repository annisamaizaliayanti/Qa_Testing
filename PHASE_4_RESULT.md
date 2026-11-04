# Phase 4 — QA Test Results

## Step 4.01 — Tech Opens My Day

**Test Scenario:** Step 4.01 — Tech Opens My Day

**App:** PWA → `/today`

| Test Step | Result |
|--------|----------|
| Login as tech | Pass — Technician successfully logged in and was redirected to My Day |
| Today's visit list displayed | Pass — Today's visit cards displayed customer name, scheduled time, and location |
| Badge/count shows job count | Pass — Job count badge displayed the correct number of visits |

---

## Step 4.02 — Tap Job → Job Detail

**Test Scenario:** Step 4.02 — Tap Job → Job Detail

**App:** PWA → `/job/:workOrderId`

| Test Step | Result |
|--------|----------|
| Tap visit card | Fail — Tapping the David Che schedule entry does not open the job detail screen |
| Hero card displayed | Not Tested — Job detail screen could not be accessed |
| Address/map link available | Not Tested — Address and map navigation could not be verified | 

---

## Step 4.03 — View Tile Grid

**Test Scenario:** Step 4.03 — View Tile Grid

**App:** PWA → Job Screen

| Tile | Expected Info |
|------|---------------|
| Scroll to tiles | Pass — All 9 tiles are visible with their initial information and counts |

---

## Step 4.04 — Read Pre-job Brief

**Test Scenario:** Step 4.04 — Read Pre-job Brief

**App:** PWA → Job Screen → "Pre-job brief"

| Test Step | Result |
|--------|----------|
| Tap "Pre-job brief" | Pass — Pre-job brief screen opened successfully for WO-0100 |
| Info displayed | Observation — The screen displays "No previous visits" and two pending sign-offs: Work Authorization Form and Customer Sign-off. Unit history, site notes, and access instructions are not displayed |

---

## Step 4.05 — Start Visit

**Test Scenario:** Step 4.05 — Start Visit

**App:** PWA → Job Screen

| Test Step | Result |
|--------|----------|
| Tap "Start visit" button | Fail — The app navigated to the Sign-offs screen instead of starting the visit |
| Auto-navigate to /job/:id/time | Fail — The Time screen did not open |
| Timer auto-starts | Not Tested — No running timer was displayed |

---

## Step 4.06 — Timer Running

**Test Scenario:** Step 4.06 — Timer Running

**App:** PWA → `/job/:id/time`

| Test Step | Result |
|--------|----------|
| Timer already running from Step 4.05 | Fail — No running timer was started after tapping Start visit |
| Entry type can be selected | Not Tested — Time screen has not been accessed | 
| Timer state persisted in localStorage key sh_timer_{visitId} | Not Tested — Timer persistence could not be verified |

---

## Step 4.07 — Open Checklist

**Test Scenario:** Step 4.07 — Open Checklist

**App:** PWA → `/job/:id/checklist`

| Test Step | Result |
|--------|----------|
| Tap Checklist tile from job screen | Pass — Checklist screen opened successfully |
| Form items displayed per job type template | Pass — All 7 checklist items are displayed, including photo, text, reading, safety, signature, and installation checklist items |
| Status per item: required/optional/ok | Pass — Required and Optional labels are displayed, and completed items show saved values or Done/Signed status |

---

## Step 4.08 — Fill Checklist — Photo Item

**Test Scenario:** Step 4.08 — Fill Checklist — Photo Item

**App:** PWA → Checklist Screen

| Test Step | Result |
|--------|----------|
| Tap photo checklist item | Camera/file picker opens |
| Upload photo | File uploaded via `uploadFile()` API |
| Form response saved | `submitFormResponses(workOrderId, { [key]: { value: fileId } })` |
| Item status → `ok` (green) | Visual feedback |

**Photo upload flow:**
1. File selected → `uploadFile({ workOrderId, kind: 'photo', checklistKey })`
2. Response saved → `submitFormResponses(workOrderId, { [checklistKey]: { value: uploadedFileId } })`
3. UI update → item shows uploaded photo thumbnail

---

## Step 4.08 — Fill Checklist — Photo Item

**Test Scenario:** Step 4.09 — Fill Checklist — Photo Item

**App:** PWA → Checklist Screen

| Test Step | Result |
|--------|----------|
| Tap photo checklist item | Pass — Photo item was accessible |
| Upload photo | Pass — Before and After photos were successfully uploaded |
| Form response saved | Pass — Uploaded photos are displayed in the checklist |
| Item status → ok (green) | Pass — Photo items display the saved photo preview |

---

## Step 4.09 — Fill Checklist — Reading Item

**Test Scenario:** Step 4.09 — Fill Checklist — Reading Item

**App:** PWA → Checklist Screen

| Test Step | Result |
|--------|----------|
| Tap photo checklist item | Pass — Photo item was accessible |
| Upload photo | Pass — Before and After photos were successfully uploaded |
| Form response saved | Pass — Uploaded photos are displayed in the checklist |
| Item status → ok (green) | Pass — Photo items display the saved photo preview |

---

## Step 4.10 — Fill Checklist — Signature Item

**Test Scenario:** Step 4.10 — Fill Checklist — Signature Item

**App:** PWA → Checklist Screen → navigates to Sign-off screen

| Test Step | Result |
|--------|----------|
| Open customer signature item | Pass — Customer signature item was accessible |
| Draw customer signature|
| Signature preview shown | Pass — Signature preview was displayed |
| Signature response saved | Pass — Checklist displays the status Signed |
| Tap "Send for remote signing" | Observation — After selecting "Send signing link instead", the document status changed to Signed | 
| Customer receives signing link | Not Tested — No signing link delivery was confirmed |
| "Resend" button available | Not Tested — No resend option was verified |

---

## Step 4.11 — Check Progress on Job Screen

**Test Scenario:** Step 4.11 — Check Progress on Job Screen

**App:** PWA → Job Screen (`/job/:id`)

| Test Step | Result |
|--------|----------|
| Return to job screen | Pass — Returned to WO-0101 Job Detail screen |
| Checklist tile | Pass — All 7 checklist items are complete; all required items are done |
| Photos tile | Pass — Before and After photos are uploaded and accessible |
| Time tile | Pass — Site Labor timer was previously observed running at 00:00:30 |

---

## Step 4.12 — Take Before/After Photos

**Test Scenario:** Step 4.12 — Take Before/After Photos

**App:** PWA → `/job/:id/photos`

| Test Step | Result |
|--------|----------|
| Tap Photos tile | Pass — Successfully navigated to the Photos screen |
| Sections displayed | Pass — Before, After, and Other photos sections are displayed |

---

## Step 4.13 — Stop Timer → Time Entry Created

**Test Scenario:** Step 4.13 — Stop Timer → Time Entry Created

**App:** PWA → `/job/:id/time`

| Test Step | Result |
|--------|----------|
| Tap "Stop timer" | Pass — Timer stopped after manually tapping "Stop timer & save" |
| localStorage key sh_timer_{visitId} cleared | Not Tested — Browser storage was not inspected directly |
| Start time rounded down 15 min | Not Tested — 15-minute rounding was not verified |
| End time rounded up 15 min | Not Tested — 15-minute rounding was not verified |
| Time entry auto-created | Pass — Time entries appeared after manually stopping the timer |
| Entry appears in list | Pass — Two time entries appeared with time, duration, and billable status |

## Step 4.14 — Send Message to Customer

**Test Scenario:** Step 4.14 — Send Message to Customer

**App:** PWA → Job Screen → Messages tile or `/job/:id/wmsg`

| Test Step | Result |
|--------|----------|
| Tap Messages tile | Pass — Message screen opened successfully |
| Quick message "On our way" or custom message | Pass — Quick message and custom message options were available |
| Send message | Pass — Message was sent successfully to the customer |
| Message recorded | Pass — Sent message appeared in the conversation history |

---

## Step 4.15 — Final Check — All Tiles Updated

**Test Scenario:** Step 4.15 — Final Check — All Tiles Updated

**App:** PWA → Job Screen

| Tile | Expected State |
|------|----------------|
| Checklist "0 required open" |  Pass — All required checklist items are completed |
| Photo "2 photos" (before + after) | Pass — Before and After photos are available |
| Time "1 entry — Xh Xm" | Pass — Time entries are displayed |
| Messages "1 sent" | Pass — Sent message is recorded in the conversation history |
| Sign-offs	Status per category (pre-work signed or pending) | Pass — Sign-off statuses are displayed |
| Materials	Line items from WO | Pass — Work order line items are displayed |
| Equipment	Linked equipment (if any) | Pass — No equipment is linked to this site; the empty state is displayed correctly |
| Expenses	"0" (none yet) | Pass — No expenses have been added |
| Files	Uploaded files count | Pass — Uploaded files are displayed | 