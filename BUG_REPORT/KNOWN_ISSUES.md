# StrongHeron QA — Bug Reports & Known Issues

**Application:** StrongHeron  
**Testing Type:** Manual Testing  
**Test Coverage:** Phase 1–6  
**Status:** Work in Progress  

---

## 1. Bug Report Summary

| ID | Phase | Area | Findings | Status |
|---|---|---|---|---|
| BUG-001 | 1.03 | Company Settings | Selecting Company Icon Image does not update the company icon. The letter "S" is still displayed | Fail |
| BUG-002 | 1.03 | Company Settings | Phone field accepts alphabetic characters | Fail |
| BUG-003 | 1.07 | Items | AC Unit item is saved and its detail page opens, but the item does not appear in the Items table | Fail |
| BUG-004 | 1.15 | Pricing | Installation Labor is displayed as $0.00 despite the Preferred Price Level being selected | Fail |
| BUG-005 | 2.02 | Leads | Lead Source is displayed as Unknown when the source is Web Form | Fail |
| BUG-006 | 2.03 | Leads | Lead status remains New after an SMS is successfully sent. Expected status: Contacted | Fail |
| BUG-007 | 3.03 | Quote Items | AC Unit price is displayed as $250 instead of the expected $2,500 | Fail |
| BUG-008 | 3.03 | Quote Items | Installation Labor is displayed as $250 instead of the expected Preferred Price Level of $450 | Fail |
| BUG-009 | 3.04 | Quote Items | Loyalty Discount is unavailable in the item selection | Fail |
| BUG-010 | 3.05 | Quote Record | Required Deposit configuration and Deposit Percentage are missing from the Quote Record | Fail |
| BUG-011 | 3.07 | Customer Quote Portal | Line items displayed in the customer portal do not match the expected quote items | Fail |
| BUG-012 | 3.07 | Customer Quote Portal | Loyalty Discount is not displayed as an optional add-on checkbox | Fail |
| BUG-013 | 3.07 | Customer Quote Portal | Quote total is displayed as $275 instead of the expected $3,025 | Fail |
| BUG-014 | 3.07 | Customer Quote Portal | Deposit amount is displayed as $0 instead of the expected $1,512.50 | Fail |
| BUG-015 | 3.10 | Customer Quote Portal | Card payment option is unavailable in the Quote Portal | Fail |
| BUG-016 | 3.11 | Approved Quote | Approved quote does not contain the expected AC Unit, Installation Labor, and Trip Fee items | Fail |
| BUG-017 | 4.02 | My Day | Selecting David Che's schedule in My Day does not open the Job Detail page | Fail |
| BUG-018 | 4.04 | Pre-job Brief | Pre-job Brief does not display Unit History, Site Notes, and Access Instructions | Fail |
| BUG-019 | 4.05 | Start Visit | Selecting Start Visit navigates to Sign-offs instead of the Time section | Fail |
| BUG-020 | 4.06 | Time Tracking | Timer does not start automatically after selecting Start Visit | Fail |
| BUG-021 | 4.10 | Customer Sign-off | After selecting "Send signing link instead", the document status immediately changes to Signed. Actual link delivery and customer signing were not verified | Observation |
| BUG-022 | 5.06 | Document Signing | Review & Sign link displays "Link unavailable" with an unexpected HTML response | Fail |
| BUG-023 | 5.07 | Signed PDF | Signed PDF displays Work Order details and customer confirmation, but does not include line items or totals | Fail |
| BUG-024 | 5.09 | Job Completion | Selecting Complete Job displays the error "Could not complete job — please try again" | Fail |
| BUG-025 | 5.10 | Work Order Status | WO-0100 status is displayed as Invoiced instead of Complete | Fail |
| BUG-026 | 5.12 | Invoice Creation | Invoice creation is blocked with the message "Work order must be complete before creating an invoice" | Fail |
| BUG-027 | 6.07 | Start Visit | After the deposit is paid, the Start Visit button is available, but selecting it opens Documents instead of Time | Fail |
| BUG-028 | 6.09 | Photo Upload | The same photo uploaded as Before automatically appears in After and cannot be removed from the After section | Fail |
| BUG-029 | 6.11 | Checkout | Checkout opens even though the total amount exceeds the authorized NTE amount | Fail |
| BUG-030 | 6.11 | Checkout | The "Costs exceed authorized amount" warning is not displayed on the Checkout screen | Fail |
| BUG-031 | 6.11 | Payment | Collect $2,200 button remains available even though the total exceeds the authorized NTE amount | Fail |
| BUG-032 | 6.12 | Change Authorization | The document list displays an additional $300, but the Review document body does not explicitly explain the additional amount | Observation |
| BUG-033 | 6.15 | Checkout | Balance Due is displayed as $2,200 instead of the expected $2,700 after a $500 deposit from a $3,200 total | Fail |

---

## 2. Not Tested / Blocked Summary

| ID | Phase | Area | Findings | Status |
|---|---|---|---|---|
| NT-001 | 3.08 | Optional Add-on | The impact of selecting an optional add-on on the quote total and deposit could not be tested because Loyalty Discount was unavailable | Not Tested |
| NT-002 | 3.11 | Deposit Payment | Deposit payment verification could not be performed because the expected deposit configuration and payment flow were unavailable | Blocked |
| NT-003 | 4.05–4.06 | Visit and Timer | Timer testing was affected by the Start Visit navigation issue | Blocked |
| NT-004 | 4.10 | Customer Sign-off | Actual signing link delivery and customer signing could not be independently verified | Not Tested |
| NT-005 | 5.09–5.10 | Job Completion | Completion timestamp and visit status could not be independently verified because of the job completion issue | Blocked |
| NT-006 | 5.12–5.13 | Invoice Workflow | Invoice review, sending, locking, customer portal payment, and notification could not be tested because the invoice draft was not created | Blocked |
| NT-007 | 6.02 | Work Order Status | PATCH status update was not performed because backend/API access was unavailable | Not Tested |
| NT-008 | 6.08 | Document Template | Document template creation through the backend/API could not be performed because backend/API access was unavailable | Not Tested |
| NT-009 | 6.13 | NTE Override | NTE override testing required backend/API access, which was unavailable | Blocked |
| NT-010 | 6.13 | NTE Override Verification | Further verification after the Admin override was not performed | Not Tested |

---

## 3. Documentation Issues

| ID | Phase | Area | Findings | Status |
|---|---|---|---|---|
| DOC-001 | Phase 4 | Step Numbering | Step 4.08 appears twice in the documentation | Observation |
| DOC-002 | Phase 4 | Step Title | Step 4.09 is titled "Reading Item", but the test case table describes Photo Item | Observation |
| DOC-003 | Phase 5 | Phase Heading | The Phase 5 section heading incorrectly refers to Phase 2 | Observation |
| DOC-004 | Phase 6 | Step Numbering | Step 6.01 and Step 6.05 headings/order are inconsistent in the documentation | Observation |

---

## 4. Status Definitions

| Status | Description |
|---|---|
| Fail | Actual application behavior does not match the expected result |
| Observation | Unexpected behavior was observed and requires further investigation |
| Not Tested | The test case was not performed |
| Blocked | Testing could not proceed because of an access limitation, dependency, or previous issue |

---

## 5. Additional Notes

1. BUG-024 and BUG-026 are related but refer to different functions: job completion and invoice creation.
2. BUG-021 records an unexpected status change to Signed. It does not confirm that the customer actually signed the document.
3. BUG-032 is recorded as an observation because the additional amount is displayed in the document list but is not explicitly explained in the Review document body.
4. Screenshots are included only for test cases where evidence was captured.
5. Not Tested and Blocked cases are documented separately from confirmed failures.
6. This report covers the documented manual testing activities in Phase 1–6 only.
7. Phase 7–14 are not included in this report.

---

**End of Report**