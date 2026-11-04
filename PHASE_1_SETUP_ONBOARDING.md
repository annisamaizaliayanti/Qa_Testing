# Phase 1 — Setup & Onboarding Flow

> **Concept:** A new company registers and configures the entire system before starting operations.
> Admin logs in, configures the company, creates roles, job types, checklists, items catalog, invites the team, and creates the first customer.
> After this phase is complete, the system is ready to receive leads and create work orders.

---

## QA Test Case Summary

| User Story | Test Scenario | Test Step | Result | Status |
|---|---|---|---|---|
| User sets up and onboards the company | TC-1.01 Admin Login | Open /login, enter admin credentials, click Login | Redirect to /dashboard, token stored in localStorage key sh_token, dashboard shows stats/approvals/quick actions |  |
| User sets up and onboards the company | TC-1.02 Dashboard Verification | View dashboard home screen | Operational stats, open approvals, exceptions, and quick actions (New WO, New Customer) all visible |  |
| User sets up and onboards the company | TC-1.03 Company Settings | Fill Identity (name, work type, icon), Address, Contact fields → Save | Data persisted, survives page reload |  |
| User sets up and onboards the company | TC-1.04 Create Custom Role | Add role "Senior Tech" with Technician base + quotes:view permission | Role appears in roles table with correct permission set |  |
| User sets up and onboards the company | TC-1.05 Job Types & Statuses | Add job types "AC Install" and "Maintenance" | Types appear in list and available in WO form dropdown |  |
| User sets up and onboards the company | TC-1.06 Forms & Checklists | Create "AC Install Checklist" template with 3 required items (photo, reading, signature), set REQUIRED AT = checkout | Template saved, items listed, will trigger at tech checkout |  |
| User sets up and onboards the company | TC-1.07 Items Catalog | Create 4 items: AC Unit ($2,500), Installation Labor ($500), Trip Fee ($75), Loyalty Discount (-$50) | Each item saved with IT_ key prefix, all 4 visible in items table |  |
| User sets up and onboards the company | TC-1.08 Price Levels | Set Standard ($500), Preferred ($450), Member ($400) on Installation Labor | 3 levels saved; Preferred customer auto-uses $450 rate |  |
| User sets up and onboards the company | TC-1.09 Invite Technician | Send invite to tech@strongheron.com with role "Technician" | Member created with status=invited, seat consumed |  |
| User sets up and onboards the company | TC-1.10 Accept Invite | Open invite link, set password | Member status changes from invited → active |  |
| User sets up and onboards the company | TC-1.11 Tech Login to PWA | Login at PWA /login with tech credentials | Redirect to /today (My Day), empty schedule shown |  |
| User sets up and onboards the company | TC-1.12 Create Customer | Fill Customer Info, Contacts (primary), Billing, Billing Address, Sites, External Refs → Save | Customer saved with CUS_ key, all sections populated in record page |  |
| User sets up and onboards the company | TC-1.13 Add Second Site | Click "+ Site" on customer record, fill address | Customer shows 2 sites in sites list |  |
| User sets up and onboards the company | TC-1.14 Add Contact | Click "+ Contact", fill name/phone/email, set as primary | Contact linked to customer with primary badge |  |
| User sets up and onboards the company | TC-1.15 Set Price Level | Set customer price level to "Preferred" in Billing section | Saved; future quotes/WOs for this customer use Preferred pricing |  |

**Reference:** [StrongHeron QA Test Cases.md](../../StrongHeron%20QA%20Test%20Cases.md) — Phase 1

---

## Step 1.01 — Admin Login

**Test Scenario:** Step 1.01 — Admin Login

**App:** Admin (localhost:5173)

| Test Step | Result |
|--------|----------|
| Open `/login` | Login page displayed |
| Enter email + password | Token stored in `localStorage` key `sh_token` |
| Click Login | Redirect to `/dashboard` |
| Dashboard displayed | Stats, approvals, exceptions, quick actions visible |

---

## Step 1.02 — Check Dashboard

**Test Scenario:** Step 1.02 — Check Dashboard

**App:** Admin → `/dashboard`

| Test Step | Result |
|--------|----------|
| View Dashboard home screen | Operational stats, open approvals, exceptions displayed |
| Quick actions available | Shortcuts for New WO, New Customer, etc. |

---

## Step 1.03 — Setup Company Settings

**Test Scenario:** Step 1.03 — Setup Company Settings

**App:** Admin → Settings → Company details

**Form fields to fill:**

### Identity
| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| COMPANY NAME | input text | ✅ | "StrongHeron HVAC" |
| WORK TYPE | select | - | Options: HVAC, Plumbing, Electrical, General |
| COMPANY ICON | radio (Letter / Image) | - | Letter or upload image |

### Address
| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| STREET ADDRESS | input text | - | "123 Main St" |
| CITY | input text | - | "Springfield" |
| STATE / PROVINCE | input text | - | "CA" |
| ZIP / POSTAL | input text | - | "90210" |
| COUNTRY | input text | - | "United States" |

### Contact
| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| PHONE | input tel | - | "+1 555 000 0000" |
| EMAIL | input email | - | "hello@strongheron.com" |
| WEBSITE | input url | - | "https://strongheron.com" |
| TAX ID / ABN / GST | input text | - | "12-3456789" |

| Test Step | Result |
|--------|----------|
| Fill all fields → Save | Data saved, success toast |
| Reload page | Data persists |

---

## Step 1.04 — Setup Roles

**Test Scenario:** Step 1.04 — Setup Roles

**App:** Admin → Settings → Roles

| Test Step | Result |
|--------|----------|
| Click "Add role" | Modal opens |
| Enter ROLE NAME: "Senior Tech" | Required field |
| Set permissions (based on Technician, add quotes=view) | Permission checkboxes per module |
| Save | New role appears in table |

**Permission modules:** Customers, Work Orders, Quotes, Invoices, Payments, Items, Equipment, Team, Reports, Roles, Value Sets, Field Defs, Expenses

---

## Step 1.05 — Setup Job Types & Statuses

**Test Scenario:** Step 1.05 — Setup Job Types & Statuses

**App:** Admin → Settings → Job types & statuses

| Test Step | Result |
|--------|----------|
| Add job type "AC Install" | Appears in list |
| Add job type "Maintenance" | Appears in list |
| Job types available in Work Order form | Selectable in dropdown |

---

## Step 1.06 — Setup Forms & Checklists

**Test Scenario:** Step 1.06 — Setup Forms & Checklists

**App:** Admin → Settings → Forms & checklists

**Form template fields:**
| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| DOCUMENT NAME | input text | ✅ | "AC Install Checklist" |
| KIND | select | - | Options per KIND_OPTIONS |
| ACTIVE | checkbox | - | ✅ checked |
| REQUIRED AT | select multi | - | Options: checkout, pre_work, change_auth |
| DOCUMENT BODY | textarea | - | Template text |

| Test Step | Result |
|--------|----------|
| Create template "AC Install Checklist" | Template saved |
| Add 3 required items: photo item, reading item, signature item | Items appear in template |
| Set REQUIRED AT = checkout | Template will trigger at tech checkout |

---

## Step 1.07 — Setup Items Catalog

**Test Scenario:** Step 1.07 — Setup Items Catalog

**App:** Admin → Items → New Item (`/items/new`)

**Form fields per item:**

| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| Item name | input text | ✅ | "AC Unit" |
| Item type | select | - | service, material, equipment, fee, discount |
| Category | select | - | From value-set categories |
| Unit of measure | select | - | each, hour, sqft, etc. |
| Cost | number (cents) | - | "2500.00" |
| Description | textarea | - | "Central AC unit 3 ton" |
| Taxable | select | - | yes / no |
| Creates equipment on install | select | - | yes / no |
| Warranty (months) | number | - | "12" |

**Subscription fields (if subscription item):**
| Field | Type | Example Data |
|-------|------|-------------|
| Billing amount | number | "99.00" |
| Billing frequency | select | monthly, quarterly, yearly |
| Visits per year | number | "4" |
| Member pricing | select | yes / no |

**Create 4 items:**

| Item | Type | Price |
|------|------|-------|
| AC Unit | material | $2,500.00 |
| Installation Labor | service | $500.00 |
| Trip Fee | fee | $75.00 |
| Loyalty Discount | discount | -$50.00 |

| Test Step | Result |
|--------|----------|
| Create each item → Save | Item saved with `IT_` key prefix |
| List items | All 4 items visible in table |

---

## Step 1.08 — Setup Price Levels

**Test Scenario:** Step 1.08 — Setup Price Levels

**App:** Admin → Items → Open "Installation Labor" → Price levels section

| Price Level | Amount | Notes |
|-------------|--------|-------|
| Standard | $500.00 | Default |
| Preferred | $450.00 | Loyal customer |
| Member | $400.00 | Subscriber |

| Test Step | Result |
|--------|----------|
| Enter price per level → Save | 3 levels saved |
| Customer with price level "Preferred" automatically uses $450 | Verified in Phase 3 |

---

## Step 1.09 — Invite Technician

**Test Scenario:** Step 1.09 — Invite Technician

**App:** Admin → Team → Invite (`/team/invite`)

**Form fields:**
| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| Email | input email | ✅ | "tech@strongheron.com" |
| Name | input text | - | "Alex Johnson" |
| Role | select | - | Select "Technician" or "Senior Tech" |

| Test Step | Result |
|--------|----------|
| Fill fields → Send invite | Member created, status=invited |
| Check in Team list | New member appears, seat consumed |

---

## Step 1.10 — Tech Accept Invite

**Test Scenario:** Step 1.10 — Tech Accept Invite

**App:** Admin → `/invite/:token`

| Test Step | Result |
|--------|----------|
| Open invite link from email | Set password page displayed |
| Set password → Submit | Member status changes: invited → active |
| Check in Team list | Status = active |

---

## Step 1.11 — Tech Login to PWA

**Test Scenario:** Step 1.11 — Tech Login to PWA

**App:** PWA Mobile (localhost:5175)

| Test Step | Result |
|--------|----------|
| Open `/login` in PWA | Login page displayed |
| Enter tech credentials | Login successful |
| Redirect to `/today` (My Day) | Today's schedule displayed (empty — no jobs yet) |

---

## Step 1.12 — Create First Customer

**Test Scenario:** Step 1.12 — Create First Customer

**App:** Admin → Customers → New (`/customers/new`)

**Section: Customer Info**
| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| Name | input text | ✅ | "Hensley Residence" |
| Type | select | - | residential, commercial |
| Customer since (year) | number | - | "2024" |
| Lead source | select | - | From value-set lead sources |
| Tags | input text | - | "property management, priority" |
| Internal notes | textarea | - | "VIP customer" |

**Section: Contacts (repeatable)**
| Field | Type | Required | Example Data |
|-------|------|----------|-------------|
| Name | input text | ✅ | "John Hensley" |
| Title | input text | - | "Property Manager" |
| Phone | input text | - | "+1 555 123 4567" |
| Email | input email | - | "john@hensley.com" |
| External ref | input text | - | "CRM-001" |
| Is Primary | checkbox | - |  |

**Section: Billing**
| Field | Type | Example Data |
|-------|------|-------------|
| Price level | select | Standard / Preferred / Member |
| Payment terms | select | From value-set |
| Statement frequency | select | monthly, quarterly, never |
| SMS notifications | checkbox |  |
| Email notifications | checkbox |  |

**Section: Billing Address**
| Field | Type | Example Data |
|-------|------|-------------|
| Street address | input text | "456 Oak Ave" |
| Line 2 | input text | "Apt 2B" |
| City | input text | "Springfield" |
| State | input text (2 char) | "CA" |
| ZIP | input text | "90210" |

**Section: Sites (repeatable)**
| Field | Type | Example Data |
|-------|------|-------------|
| Label | input text | "Main House" |
| Street address | input text | "456 Oak Ave" |
| City, State, ZIP | input text | "Springfield, CA 90210" |
| Access notes | input text | "Gate code: 1234" |
| Gate code | input text | "1234" |
| External ref | input text | "SITE-001" |
| Is Primary | checkbox |  |

**Section: External References**
| Field | Type | Example Data |
|-------|------|-------------|
| External ref (customer) | input text | "ERP-ID-001" |

| Test Step | Result |
|--------|----------|
| Fill all sections → Save | Customer saved with `CUS_` key |
| Open customer record | All data displayed on record page |

---

## Step 1.13 — Add Site to Customer

**Test Scenario:** Step 1.13 — Add Site to Customer

**App:** Admin → Customer Record → Sites section

| Test Step | Result |
|--------|----------|
| Click "+ Site" | Site form fields appear |
| Fill second site address | Site linked to customer |
| Customer has 2 sites | Sites list shows 2 entries |

---

## Step 1.14 — Add Contact

**Test Scenario:** Step 1.14 — Add Contact

**App:** Admin → Customer Record → Contacts section

| Test Step | Result |
|--------|----------|
| Click "+ Contact" | Contact form fields appear |
| Fill name, phone, email | Contact linked to customer |
| Set as primary contact | Primary badge displayed |

---

## Step 1.15 — Set Customer Price Level

**Test Scenario:** Step 1.15 — Set Customer Price Level

**App:** Admin → Customer Record → Billing section

| Test Step | Result |
|--------|----------|
| Set Price level = "Preferred" | Saved |
| When creating quote/WO for this customer | Price automatically uses Preferred level ($450 for Installation Labor, not $500) |

---

## ✅ Phase 1 Results

| Item | Status |
|------|--------|
| Admin login & dashboard |  |
| Company settings configured |  |
| Roles created (Senior Tech) |  |
| Job types created (AC Install, Maintenance) |  |
| Checklist template created (3 required items) |  |
| Items catalog (4 items + price levels) |  |
| Technician invited & activated |  |
| Tech login to PWA successful |  |
| First customer + sites + contacts |  |
| Price level set |  |

**System is ready for Phase 2: receive leads and create work orders.**
