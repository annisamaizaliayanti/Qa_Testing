# Phase 1 — QA Test Results

## Step 1.01 — Admin Login

**Test Scenario:** Step 1.01 — Admin Login

**App:** Admin (localhost:5173)


| Test Step | Result |
|--------|----------|
| Open/Login | Pass - Login page displayed |
| Enter email + password | Pass - Admin credentials accepted |
| Click Login | Pass - Login successful |
| Dashboard displayed | Pass - Admin dashboard displayed |

---

## Step 1.02 - Check Dashboard

**Test Scenario:** Step 1.02 — Check Dashboard

**App:** Admin → `/dashboard`

| Test Step | Result |
|--------|----------|
| View Dashboard home | Pass - Operational stats, pending approvals, and expections displayed |
| Quick actions available | Pass - Shortcuts for New Work Order and other available displayed

---

## Step 1.03 — Setup Company Settings

**Test Scenario:** Step 1.03 — Setup Company Settings

**App:** Admin → Settings → Company details

### Identity

| Test Step | Result |
|--------|----------|
| Company Name | Pass - Company Name successfully entered |
| Select Work Type | Pass - HVAC successfully selected|
| Select Company Icon - Letter | Pass - Leter option successfully selected |
Select Company Icon - Image | Fail - Image options selected, but company profile still displays the letter "S" |

### Address

| Test Step | Result |
|--------|----------|
| Enter Street Address | Pass - Street address successfully entered|
| Enter City | Pass - City successfully entered |
| Enter State / Province | Pass - State  / Province successfully entered |
| Enter ZIP / Postal | Pass - ZIP / Postal successfully entered |
| Enter Country | Pass - Country successfully entered |

### Contact

| Test Step | Result |
|--------|----------|
| Enter Phone | Fail - Phone field accepts alphabetic characters |
| Enter Email | Pass - Email successfully entered |
| Enter Tax ID / ABN / GST |  Pass - Tax ID successfully entered |

### Save & Reload

| Test Step | Result |
|--------|----------|
| Fill all fields -> Save | Pass - Company details saved successfully; "saved" notification displayed |
| Reload page | Pass - Company details persisted after page reload|

---

## Step 1.04 — Setup Roles

**Test Scenario:** Step 1.04 — Setup Roles

**App:** Admin → Settings → Roles

| Test Step | Result |
|--------|----------|
| Click "Add role" | Pass - Permissions page opened successfully |
| Enter ROLE NAME: "Senior Tech" | Pass - Role name Senior Tech displayed succesfully |
| Set permissions (based ono Technician, add quotes=view) | Pass - echnician default permissions loaded and Quotes permission set to View |
| Save | Pass - Permission changes saved successfully, Saved indicator displayed |

---

## Step 1.05 — Setup Job Types & Statuses

**Test Scenario:** Step 1.05 — Setup Job Types & Statuses

**App:** Admin → Settings → Job types & statuses

| Test Step | Result |
|--------|----------|
| Add job type "AC Install" | Pass - AC Install successfully added and automatically saved |
| Add job type "Maintenance" | Pass - Maintenance successfully added and automatically saved |
| Job types available in Work Order form | Pass - Job types successfully appeared and could be selected from the dropdown |

---

## Step 1.06 — Setup Forms & Checklists

**Test Scenario:** Step 1.06 — Setup Forms & Checklists

**App:** Admin → Settings → Forms & checklists

| Test Step | Result |
|--------|----------|
| Create template "AC Install Checklist" | Pass - Template successfully created and saved |
| Add 3 required items: photo item, reading item, signature item | Pass - Photo, Reading, and Signature items have been successfully selected |
| Set REQUIRED AT = checkout | Pass - Checkout selected as the required stage and template successfully saved |

---

## Step 1.07 — Setup Items Catalog

**Test Scenario:** Step 1.07 — Setup Items Catalog

**App:** Admin → Items → New Item (`/items/new`)

| Test Step | Result |
|--------|----------|
| Create each item → Save | Pass - Item successfully saved with code ITEM-0029 |
| List Item | Fail — AC Unit is accessible on its detail page but does not appear in the Items table |

---

## Step 1.08 — Setup Price Levels

**Test Scenario:** Step 1.08 — Setup Price Levels

| Test Step | Result |
|--------|----------|
| Enter price per level → Save | Pass — 3 price levels saved (Standard $500, Preferred $450, Member $400) |
| Customer with price level "Preferred" automatically uses $450 | Not Tested — To be verified in Phase 3 |

---

## Step 1.09 — Invite Technician

**Test Scenario:** Step 1.09 — Invite Technician

| Test Step | Result |
|--------|----------|
| Fill fields → Send invite | Pass - Member successfully created with the status 'invited' |
| Check in Team list | Pass - A new member appears on the list, and a seat is taken |

---

## Step 1.10 — Tech Accept Invite

**Test Scenario:** Step 1.10 — Tech Accept Invite

**App:** Admin → `/invite/:token`

| Test Step | Result |
|--------|----------|
| Open invite link from email | Pass - Set password page displayed |
| Set password → Submit | Pass - Member status changes from invited to active |
| Check in Team list | Pass - Member status displayed as active |

---

**Test Scenario:** Step 1.11 — Tech Login to PWA

**App:** PWA Mobile (localhost:5175)

| Test Step | Result |
|--------|----------|
| Open / login PWA | Pass - Login page displayed |
| Enter tech credentials | Pass - Technician successfully logged in |
| Redirect to /today (My Day) | Pass - edirected to My Day and today's schedule displayed (empty — no jobs yet) |

---

## Step 1.12 — Create First Customer

**Test Scenario:** Step 1.12 — Create First Customer

**App:** Admin → Customers → New (`/customers/new`)

| Test Step | Result |
|--------|----------|
| Fill all sections → Save | Pass — Customer successfully saved with a key starting with CUS_ |
| Open customer record | Pass — All data successfully displayed on the customer record page |

---

## Step 1.13 — Add Site to Customer

**Test Scenario:** Step 1.13 — Add Site to Customer

**App:** Admin → Customer Record → Sites section

| Test Step | Result |
|--------|----------|
| Click "+ Site" | Pass — Site form fields appeared successfully |
| Fill second site address | Pass — Second site successfully added and linked to the customer |
| Customer has 2 sites | Pass — Sites list displays 2 entries |

---

## Step 1.14 — Add Contact

**Test Scenario:** Step 1.14 — Add Contact

**App:** Admin → Customer Record → Contacts section

| Test Step | Result |
|--------|----------|
| Click "+ Contact" | Pass — Contact form fields appeared successfully |
| Fill name, phone, email | Pass — Contact successfully added and linked to the customer |  
| Set as primary contact | Pass — Primary badge displayed on the contact |

---

**Test Scenario:** Step 1.15 — Set Customer Price Level

**App:** Admin → Customer Record → Billing section

| Test Step | Result |
|--------|----------|
| Set Price level = "Preferred" | Pass — Price level successfully saved as Preferred |
| When creating quote/WO for this customer | Fail — Installation Labor displayed $0.00 instead of the expected Preferred price of $450 |