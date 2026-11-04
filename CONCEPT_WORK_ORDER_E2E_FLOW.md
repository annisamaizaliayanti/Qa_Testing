# Konsep Lengkap: Work Order End-to-End Flow

> Dari mockup `AppMockup.html` — bagaimana Work Order bekerja dari awal sampai akhir,
> semua tab, semua relasi, dan flow setiap aktor.

---

## 1. GAMBARAN BESAR — Apa Itu Work Order?

Work Order (WO) adalah **unit pekerjaan di lapangan**. Semua yang dikerjakan tech tercatat di sini:
waktu kerja, material yang dipakai, foto, checklist, pembayaran, dan dokumen.

WO bisa berdiri sendiri (service call biasa) atau menjadi bagian dari **Project** yang lebih besar.

```
ASAL WO:
├── Manual       → Admin buat langsung dari menu Work Orders → New
├── Dari Lead    → Convert lead → buat WO
├── Dari Quote   → Quote approved → WO auto-terbuat
├── Dari Project → Admin buat dari Project Record → "New work order"
└── Dari Agreement → Service plan generate recurring WO
```

---

## QA Test Case Summary

| User Story | Test Scenario | Test Step | Result | Status |
|---|---|---|---|---|
| User menjalankan WO dari awal sampai payout (standalone / dari quote / dari project) | Flow 1 — Buat WO manual | Admin → Work Orders → New, isi Customer, Site, Job type, Tech, Schedule, Lines | WO_xxx terbuat, status "new" (lihat TC-2.07) |  |
| User menjalankan WO dari awal sampai payout (standalone / dari quote / dari project) | Flow 1 — Schedule & Dispatch | Admin set tanggal + waktu + tech | Visit terbuat, muncul di dispatch board, status → scheduled (lihat TC-2.08) |  |
| User menjalankan WO dari awal sampai payout (standalone / dari quote / dari project) | Flow 1 — Tech mulai kerja (PWA) | Tech buka My Day, tap "Start visit" | Status → in_progress, timer otomatis jalan (lihat TC-4.05) |  |
| User menjalankan WO dari awal sampai payout (standalone / dari quote / dari project) | Flow 1 — Tech lengkapi checklist | Complete photo/reading/signature items, isi install gate jika ada creates_equipment | Semua required items done (lihat TC-4.07–TC-4.10, TC-7.06) |  |
| User menjalankan WO dari awal sampai payout (standalone / dari quote / dari project) | Flow 1 — Tech catat time | Timer auto-stop atau manual entry (type, duration, billable, cost code) | Entry muncul di WO → tab Time (lihat TC-4.13, TC-12.02–TC-12.03) |  |
| User menjalankan WO dari awal sampai payout (standalone / dari quote / dari project) | Flow 1 — Tech catat materials | Tambah item line tambahan dari catalog | Muncul di WO → tab Materials (lihat TC-5.02) |  |
| User menjalankan WO dari awal sampai payout (standalone / dari quote / dari project) | Flow 1 — Tech catat expenses | Input vendor, amount, receipt, billable flag, submit | Muncul di WO → tab Expenses, masuk approval queue (lihat TC-11.01–TC-11.02) |  |
| User menjalankan WO dari awal sampai payout (standalone / dari quote / dari project) | Flow 1 — Tech ambil photos | Upload before/after photos, tag customer-visible jika perlu | Foto muncul di WO → tab Photos (lihat TC-4.12) |  |
| User menjalankan WO dari awal sampai payout (standalone / dari quote / dari project) | Flow 1 — Tech checkout | Review totals, customer sign, collect payment, complete job | WO status → complete, visit completed (lihat TC-5.01–TC-5.09) |  |
| User menjalankan WO dari awal sampai payout (standalone / dari quote / dari project) | Flow 1 — Server-side on complete | POST /work-orders/:id/complete | Equipment auto-created jika ada creates_equipment line, signed documents generated (lihat TC-5.10, TC-7.08–TC-7.09) |  |
| User menjalankan WO dari awal sampai payout (standalone / dari quote / dari project) | Flow 1 — Admin create & send invoice | Create invoice dari WO lines + billable expenses + billable time | Invoice draft → sent, status open (lihat TC-5.12–TC-5.13) |  |
| User menjalankan WO dari awal sampai payout (standalone / dari quote / dari project) | Flow 1 — Customer bayar | Customer buka portal invoice, bayar remaining balance | Invoice status → paid, balance $0 (lihat TC-8.05–TC-8.07) |  |
| User menjalankan WO dari awal sampai payout (standalone / dari quote / dari project) | Flow 2 — WO dari Quote approved | Quote approved + deposit dibayar | Deposit invoice + payment auto-created, WO auto-created dari quote (created_from=quote) (lihat TC-3.10–TC-3.13) |  |
| User menjalankan WO dari awal sampai payout (standalone / dari quote / dari project) | Flow 3 — WO dari Project | Admin buat WO dari Project Record → "New work order", pilih phase | projectId & customerId auto-filled, WO linked ke phase (lihat TC-10.05, TC-10.11) |  |
| User menjalankan WO dari awal sampai payout (standalone / dari quote / dari project) | Gap — Project/Phase field di WO New form | Buka form "New work order" standalone (bukan dari Project Record) | Field Project & Phase/WBS belum tersedia di form | ❌ Belum ada |
| User menjalankan WO dari awal sampai payout (standalone / dari quote / dari project) | Gap — Crew & header actions (Text customer, Download PDF) | Cek header WO Record | Status badge, provenance link, Text customer, Download PDF belum ada di implementasi | ❌ Belum ada |

**Reference:** Lihat TC-ID terkait di [PHASE_2](PHASE_2_LEAD_TO_WO.md), [PHASE_3](PHASE_3_LEAD_TO_QUOTE_TO_WO.md), [PHASE_4](PHASE_4_TECH_JOB_EXECUTION.md), [PHASE_5](PHASE_5_CHECKOUT_PAYMENT.md), [PHASE_7](PHASE_7_EQUIPMENT_INSTALL.md), [PHASE_8](PHASE_8_INVOICE_LIFECYCLE.md), [PHASE_10](PHASE_10_PROJECT_MULTIPHASE.md), [PHASE_11](PHASE_11_EXPENSES_FLOW.md), [PHASE_12](PHASE_12_TIME_ENTRY_FLOW.md)

---

## 2. ENTITY RELATIONSHIP

```
                   ┌───────────┐
                   │  CUSTOMER │
                   └─────┬─────┘
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
    ┌──────────┐  ┌──────────┐   ┌──────────┐
    │   LEAD   │  │  QUOTE   │   │ PROJECT  │
    └────┬─────┘  └────┬─────┘   └────┬─────┘
         │             │              │
         └──────┬──────┘              │
                ▼                     ▼
         ┌─────────────────────────────────┐
         │          WORK ORDER             │
         │  WO_214 · RTU install           │
         │  Status: new → scheduled →      │
         │          in_progress → complete  │
         └───────────────┬─────────────────┘
                         │
     ┌──────┬──────┬─────┼──────┬──────┬──────┬──────┐
     ▼      ▼      ▼     ▼      ▼      ▼      ▼      ▼
  VISITS  TIME  MATERIALS EXPENSES PHOTOS PAYMENTS FILES  FORMS
  (schedule)(jam  (items  (beli    (foto  (deposit (docs (checklist
   + tech) kerja) lines)  lapangan) bukti) + final) surat) items)
                         │
                         ▼
                   ┌──────────┐
                   │ INVOICE  │──→ PAYMENT ──→ PAYOUT
                   └──────────┘
```

---

## 3. WORK ORDER RECORD — 11 TAB (dari mockup)

### Tab yang ada di mockup:
```
Overview | Visits | Time | Materials | Forms & checklists | Photos | Payments | Files | Messages | Related | Activity
```

### Tab yang ada di implementasi:
```
Overview | Visits | Time | Materials | Expenses | Photos | Payments | Forms & checklists | Sign-offs | Files | Messages | Related | Activity
```

**Perbedaan:** Implementasi punya 2 tab extra: **Expenses** dan **Sign-offs**

---

## 4. PENJELASAN SETIAP TAB

### TAB 0: Overview

**Konsep:** Ringkasan semua informasi WO dalam satu halaman — siapa customernya, dimana, kapan, berapa, dan status apa.

**Sections di mockup:**

#### DETAILS
| Field | Contoh | Deskripsi |
|-------|--------|-----------|
| Customer | Northline School District | Siapa yang dilayani |
| Site | Bldg A · 1400 Northline Rd | Lokasi kerja |
| Job type | RTU install | Jenis pekerjaan (dari value-set) |
| Priority | Standard | Prioritas (dari value-set) |
| Assigned tech | Sam Ortiz | Tech utama |
| Crew | Crew A | Tim (jika multi-tech) |

#### SCHEDULE
| Field | Contoh |
|-------|--------|
| Scheduled date | 09/08/2026 |
| Scheduled start time | 1:30 PM |
| Scheduled end time | 4:00 PM |

#### RELATED (ringkasan link)
| Type | Record |
|------|--------|
| Quote | Q_118 |
| Project | — (atau PR_07 jika linked) |
| Equipment | RTU-2 · Bldg A roof |

#### TOTALS
| Field | Amount |
|-------|--------|
| Subtotal | $4,120.00 |
| Tax | $0.00 |
| Total | $4,120.00 |

#### DEPOSIT
| Field | Value |
|-------|-------|
| Deposit required | Yes |
| Deposit amount | $1,236.00 |
| Deposit invoice | INV-1043 |
| Deposit paid | Yes |

#### BALANCE
| Field | Amount |
|-------|--------|
| Paid to date | $1,236.00 |
| Balance due | $2,884.00 |

#### LINES (item lines dari WO)
| ITEM | DESCRIPTION | QTY | RATE | TOTAL |
|------|-------------|-----|------|-------|
| RTU replacement · 5 ton | Remove and replace rooftop unit; disposal included | 1 | $3,740.00 | $3,740.00 |
| Startup & commissioning | Startup readings, verification, owner walkthrough | 1 | $380.00 | $380.00 |

---

### TAB 1: Visits

**Konsep:** Daftar semua kunjungan tech ke site untuk WO ini. Satu WO bisa punya beberapa visit (pertama datang diagnosa, kedua datang install, dll).

| VISIT | DATE | START | END | TECH | CREW | STATUS | OUTCOME |
|-------|------|-------|-----|------|------|--------|---------|
| 1 | 09/08/2026 | 1:30 PM | 4:00 PM | Sam Ortiz | Crew A | Scheduled | — |

**Actions:** + Schedule visit

**Flow:**
1. Admin schedule visit (tanggal, waktu, tech)
2. Visit muncul di dispatch board
3. Tech lihat di PWA → My Day
4. Tech start visit → status "in_progress"
5. Tech selesai → status "done"

---

### TAB 2: Time

**Konsep:** Catatan waktu kerja semua orang di WO ini — dari timer atau manual entry. Untuk **job costing** (berapa biaya labor) dan **billing** (jam billable bisa ditagihkan).

| MEMBER | ENTRY TYPE | DATE | DURATION | BILLABLE | COST CODE | MEMO |
|--------|-----------|------|----------|----------|-----------|------|
| Sam Ortiz | Shop | 09/05/2026 | 1.0 h | — | SH-100 | Stage RTU and curb adapter |
| Sam Ortiz | Site labor | 09/08/2026 | 2.5 h | Yes | LB-200 | Install |

**Actions:** + Add entry

**Hubungan:**
- Time entry → `work_order_id` = WO ini
- Billable entries bisa di-add ke invoice sebagai labor line
- Cost code tracking untuk job costing detail

---

### TAB 3: Materials

**Konsep:** Item/material **tambahan** yang dipakai di WO ini, di luar line items awal. Ini material yang di-track terpisah dari WO lines, biasanya material yang dipinjam atau disewa.

| ITEM | DESCRIPTION | COST CODE | QTY | RATE | TOTAL |
|------|-------------|-----------|-----|------|-------|
| Crane lift | Third-party crane service, half day | EQ-400 | 1 | $450.00 | $450.00 |

**Perbedaan Materials vs Lines (di Overview):**
- **Lines (Overview):** Item utama dari quote/WO — "apa yang dijual ke customer"
- **Materials (tab):** Item tambahan yang dipakai selama kerja — "apa yang terpakai di lapangan"

**Perbedaan Materials vs Expenses:**
- **Materials:** Item dari catalog items (punya item ID, rate, cost code)
- **Expenses:** Pengeluaran ad-hoc di lapangan (vendor, amount, receipt, approval flow)

---

### TAB 4: Forms & Checklists

**Konsep:** Checklist yang harus diselesaikan tech berdasarkan job type template. Memastikan semua langkah kerja dilakukan dan didokumentasikan.

Template checklist di mockup (RTU Install v3):

| CHECKLIST ITEM | TYPE | REQUIRED | STATUS |
|---------------|------|----------|--------|
| Startup photos · before | Photo | Yes | Not started |
| Startup photos · after | Photo | Yes | Not started |
| Static pressure reading | Reading · in. wc | Yes | Not started |
| Refrigerant charge verified | Checkbox | Yes | Not started |
| Customer walkthrough signature | Signature | Yes | Not started |
| Breaker panel photo | Photo | No | Not started |
| Crane inspection notes | Text | No | Not started |

**Rules:**
- Required items harus complete sebelum WO bisa di-complete
- Photo items → file di-upload dan link ke checklist
- Signature items → digital signature capture
- Reading items → input angka (dengan unit)

---

### TAB 5: Photos

**Konsep:** Semua foto terkait WO — foto sebelum kerja, sesudah kerja, foto nameplate, foto bukti. Beberapa bisa ditandai "customer visible" untuk portal customer.

| Photo | Date | By | Visible to customer? |
|-------|------|----|---------------------|
| Site access | 09/05 | S. Ortiz | No |
| Curb · existing | 09/05 | S. Ortiz | No |
| Nameplate | 09/05 | S. Ortiz | No |
| Staged unit | 09/05 | S. Ortiz | **CUSTOMER VISIBLE** |
| Roof path | 09/05 | S. Ortiz | No |
| Crane cert | 09/05 | S. Ortiz | No |

**Actions:** Drag & drop upload, browse

---

### TAB 6: Payments

**Konsep:** Semua pembayaran yang terkait WO ini — deposit, pembayaran bertahap, final payment.

| DATE | TYPE | METHOD | AMOUNT | INVOICE | BY |
|------|------|--------|--------|---------|---|
| 09/06/2026 | Deposit | Card ••4417 | $1,236.00 | INV-1043 | Customer · portal |

**Flow payments:**
1. Deposit payment → saat quote approved (otomatis)
2. Field payment → saat tech checkout (via PWA)
3. Final payment → saat customer bayar invoice via portal

---

### TAB 7: Files

**Konsep:** Dokumen terkait WO — work authorization yang ditandatangani, sertifikat, permit, dll.

| NAME | TYPE | SIZE | BY | ADDED |
|------|------|------|----|-------|
| Signed · Work authorization.pdf | PDF | 84 KB | System | 09/08/2026 |
| Crane cert.pdf | PDF | 1.2 MB | S. Ortiz | 09/05/2026 |

**Actions:** Drag & drop upload, browse

---

### TAB 8: Messages

**Konsep:** Log komunikasi dengan customer untuk WO ini — SMS, email, call logs.

| Time | Direction | Content |
|------|-----------|---------|
| 09/07 10:21 AM | Outbound · SMS | Appointment confirmation · delivered |
| 09/06 9:02 AM | Outbound · email | Deposit receipt · INV-1043 |

**Actions:** Send SMS, Send Email, Log call

---

### TAB 9: Related Records

**Konsep:** Semua record lain yang terhubung ke WO ini — navigasi cepat ke entity terkait.

| TYPE | RECORD | STATUS |
|------|--------|--------|
| Quote | Q_118 | Approved |
| Invoice · deposit | INV-1043 | Paid |
| Signed document | Work authorization + NTE $4,500 · v2 · M. Ellis · 09/08 1:12 PM | PDF |
| Customer | Northline School District | Active |
| Equipment | RTU-2 · Bldg A roof | Warranty expired |

**Yang bisa muncul di Related:**
- Quote (asal WO)
- Invoice (deposit + final)
- Signed documents (work auth, NTE, sign-offs)
- Customer
- Equipment
- Project (jika linked)
- Agreement (jika dari service plan)

---

### TAB 10: Activity

**Konsep:** Audit trail — semua yang pernah terjadi di WO ini, urut waktu.

| Time | Who | What |
|------|-----|------|
| 09/07 10:20 AM | A. Carter | Scheduled 09/08 1:30–4:00 PM, Sam Ortiz, confirmation texted |
| 09/06 9:02 AM | System | Deposit receipt emailed |
| 09/06 9:02 AM | Customer | Deposit paid $1,236.00 · card ••4417 · portal |
| 09/05 2:15 PM | System | Deposit invoice INV-1043 issued |
| 09/05 2:14 PM | A. Carter | Created from Q_118 |

---

## 5. HEADER & ACTIONS

### Header (dari mockup)

```
Work order
WO_214 · Northline School District    [Scheduled]
Created 09/05/2026 · from Q_118 · RTU install

Buttons: [Create invoice]  [Actions ▾]  [Save]
```

**Header info yang ditampilkan:**
- WO key + customer name
- Status badge
- Created date + source (from Quote/Lead/Project) + job type
- Provenance link (Q_118 bisa diklik)

### Actions Dropdown (dari mockup)

| Action | Kondisi | Deskripsi |
|--------|---------|-----------|
| Create invoice | WO complete | Buat invoice dari WO lines |
| Schedule return visit | Selalu | Jadwalkan kunjungan balik |
| Text customer… | Selalu | Kirim SMS ke customer |
| Duplicate | Selalu | Duplikat WO |
| Download PDF | Selalu | Download WO sebagai PDF |

### Implementasi sudah punya:
| Action | Status |
|--------|--------|
| Create invoice | ✅ (tombol utama saat complete) |
| Complete WO | ✅ (tombol utama saat belum complete) |
| Duplicate |  |
| Schedule return visit | ✅ (navigate ke dispatch) |
| Create return visit | ✅ (buat WO baru dengan reason) |
| Text customer | ❌ Belum ada |
| Download PDF | ❌ Belum ada |

---

## 6. ALUR KERJA END-TO-END

### FLOW 1: WO Biasa (Standalone)

```
STEP 1: BUAT WO
├── Admin → Work Orders → + New work order
├── Isi: Customer, Site, Job type, Tech, Schedule, Lines (items)
└── Hasil: WO_xxx terbuat, status "new"

STEP 2: SCHEDULE & DISPATCH
├── Admin set tanggal + waktu + tech
├── Visit terbuat → muncul di dispatch board
├── Status: new → "scheduled"
└── SMS appointment confirmation ke customer (otomatis)

STEP 3: TECH MULAI KERJA (PWA)
├── Tech buka My Day → lihat visit card
├── Tap "Start visit" → status "in_progress"
├── Timer otomatis jalan
└── Pre-work sign-off (jika required)

STEP 4: TECH LENGKAPI CHECKLIST
├── Buka checklist → items dari template job type
├── Complete photo items → upload foto
├── Complete reading items → input angka
├── Complete signature items → tanda tangan
├── Install gate items (jika ada creates_equipment line):
│   ├── Enter serial number
│   └── Upload nameplate photo
└── Semua required items harus done

STEP 5: TECH CATAT TIME
├── Timer auto-stop → time entry terbuat
├── Atau manual entry (travel, admin, dll)
├── Entry type, duration, billable flag, cost code
└── Muncul di WO → tab Time

STEP 6: TECH CATAT MATERIALS
├── Tambah item lines tambahan (jika ada material terpakai)
├── Item dari catalog, punya rate + cost code
└── Muncul di WO → tab Materials

STEP 7: TECH CATAT EXPENSES (jika ada)
├── Beli material di lapangan (ad-hoc)
├── Input: vendor, amount, receipt, billable flag
├── Submit untuk approval
└── Muncul di WO → tab Expenses (implementasi)

STEP 8: TECH AMBIL PHOTOS
├── Upload before/after photos
├── Tag customer-visible jika perlu
└── Muncul di WO → tab Photos

STEP 9: TECH CHECKOUT
├── Stop timer (jika belum)
├── Review totals breakdown:
│   ├── Subtotal (dari WO lines)
│   ├── Deposit paid (minus)
│   └── Balance due
├── Customer tanda tangan
├── Collect payment (kartu/cash)
└── Complete job → POST /work-orders/:id/complete

STEP 10: SERVER-SIDE ON COMPLETE
├── WO status → "complete"
├── Equipment auto-created (jika ada creates_equipment line)
├── Visit status → completed
└── Signed documents generated

STEP 11: ADMIN CREATE INVOICE
├── WO (completed) → "Create invoice"
├── Invoice draft terbuat dengan:
│   ├── Lines dari WO
│   ├── Billable expenses (dengan markup)
│   └── Billable time entries (hours × rate)
├── Review → Send invoice
└── Status: draft → open

STEP 12: CUSTOMER BAYAR
├── Customer buka invoice portal
├── Bayar remaining balance
├── Invoice status → "paid"
└── Balance = $0
```

---

### FLOW 2: WO dari Quote

```
STEP 1: Lead masuk → convert ke Quote
STEP 2: Admin buat quote lines (items, deposit %)
STEP 3: Send quote ke customer
STEP 4: Customer approve + bayar deposit via portal
STEP 5: ──── OTOMATIS ────
        ├── Quote status → "approved", locked
        ├── Deposit Invoice auto-created
        ├── Deposit Payment auto-created
        └── WO auto-created from quote (lines carried over)
STEP 6-12: Sama seperti Flow 1 (dari Step 3 - tech mulai kerja)

EXTRA: WO.createdFrom = "quote"
       WO.quoteId = Q_xxx
       WO.depositInvoiceId = INV_xxx
```

---

### FLOW 3: WO dari Project

```
STEP 1: Admin buat Project (customer, contract value, WBS)
STEP 2: Admin buat WO dari Project Record → "New work order"
        ├── projectId = project ini (auto-filled)
        ├── phaseId = phase yang dipilih
        ├── customerId = customer project (auto-filled)
        └── Site, tech, lines diisi admin
STEP 3-12: Sama seperti Flow 1

EXTRA: WO.projectId = PRJ_xxx
       Time entries → roll up ke project time tab
       Expenses → roll up ke project expenses tab
       WO total → roll up ke WBS budget vs actual
```

---

## 7. MEMBUAT WO (Form New Work Order)

### Field yang ada di IMPLEMENTASI sekarang

| Section | Field | Type | Required |
|---------|-------|------|----------|
| CUSTOMER | Name | select (customers) |  |
| | Site / location | select (sites) | — |
| | + Add site (inline) | form | — |
| JOB DETAILS | Assigned tech | select (members) | — |
| | Job type | select (value-set) | — |
| | Priority | select (value-set) | — |
| | Price list | select (value-set) | — |
| | Summary / description | textarea | — |
| | Internal notes | textarea | — |
| SCHEDULE | Date | date | — |
| | Start time | time | — |
| | End time | time | — |
| LINES | Item (select) + Description + Qty + Rate | repeatable rows | — |
| TOTALS | Subtotal, Tax, Total | computed | — |

### Field yang BELUM ADA (perlu untuk project linking)

| Field | Type | Kapan muncul |
|-------|------|-------------|
| **Project** | select (projects) | Selalu (optional) |
| **Phase/WBS item** | select (phases) | Muncul jika project dipilih |
| **Equipment** | select (equipment) | Selalu (optional) |
| **Crew** | select (crews) | Selalu (optional) |

### Field yang TERISI OTOMATIS saat dari Project Record

Jika WO dibuat dari tombol "New work order" di Project Record:
- `projectId` → auto-filled, read-only
- `customerId` → auto-filled dari project customer, read-only
- Phase → bisa dipilih dari phases project ini

---

## 8. GAP: IMPLEMENTASI vs MOCKUP

### 8.1 Tab Comparison

| # | Tab Mockup | Tab Implementasi | Status |
|---|-----------|-----------------|--------|
| 0 | Overview | Overview | ✅ Ada |
| 1 | Visits | Visits | ✅ Ada |
| 2 | Time | Time | ✅ Ada |
| 3 | Materials | Materials | ✅ Ada |
| — | *(tidak di mockup)* | **Expenses** | ➕ Extra (tidak di mockup) |
| 4 | Forms & checklists | Forms & checklists | ✅ Ada |
| — | *(tidak di mockup)* | **Sign-offs** | ➕ Extra (tidak di mockup) |
| 5 | Photos | Photos | ✅ Ada |
| 6 | Payments | Payments | ✅ Ada |
| 7 | Files | Files | ✅ Ada |
| 8 | Messages | Messages | ✅ Ada |
| 9 | Related | Related | ✅ Ada |
| 10 | Activity | Activity | ✅ Ada |

**Catatan:** Implementasi punya 13 tab (2 extra: Expenses + Sign-offs) vs mockup 11 tab.

### 8.2 Overview Tab — Detail Gap

| Section | Mockup | Implementasi | Gap |
|---------|--------|-------------|-----|
| Details | Customer, Site, Job type, Priority, Tech, **Crew** | Customer, Site, Job type, Priority, Tech | **Crew** belum ada |
| Schedule | Date, Start, End | Date, Start, End | ✅ Match |
| Related (ringkasan) | Quote, **Project**, Equipment | — | ❌ Belum ada section Related di Overview |
| Totals | Subtotal, Tax, Total | ✅ Ada (stat tiles) | Format berbeda (tiles vs read-only fields) |
| Deposit | Required, Amount, Invoice, Paid | ✅ Ada | ✅ Match |
| Balance | Paid to date, Balance due | ✅ Ada | ✅ Match |
| Lines | Table (item, desc, qty, rate, total) | ✅ Ada | ✅ Match |

### 8.3 Header — Detail Gap

| Element | Mockup | Implementasi | Gap |
|---------|--------|-------------|-----|
| WO key + customer | ✅ | ✅ | Match |
| Status badge | ✅ | ❌ | Belum ada |
| Created date | ✅ | ✅ (stat tile) | Format berbeda |
| "from Q_118" provenance | ✅ | ❌ | Belum ada di header |
| Job type label | ✅ | ❌ | Belum ada di header |
| Create invoice button | ✅ | ✅ | Match |
| Complete WO button | — | ✅ | Extra |
| Actions: Duplicate | ✅ | ✅ | Match |
| Actions: Schedule return | ✅ | ✅ | Match |
| Actions: Text customer | ✅ | ❌ | Belum ada |
| Actions: Download PDF | ✅ mockup | ❌ | Belum ada |
| Save button | ✅ mockup | ❌ | Inline edit instead |

### 8.4 WO New Form — Gap

| Field | Mockup (inferred) | Implementasi | Gap |
|-------|-------------------|-------------|-----|
| Customer | ✅ | ✅ | Match |
| Site | ✅ | ✅ | Match |
| Job type | ✅ | ✅ | Match |
| Priority | ✅ | ✅ | Match |
| Tech | ✅ | ✅ | Match |
| **Crew** | ✅ (di WO record) | ❌ | Belum ada |
| **Project** | ✅ (related) | ❌ | Belum ada |
| **Equipment** | ✅ (related) | ❌ | Belum ada |
| Price list | — | ✅ | Extra |
| Schedule | ✅ | ✅ | Match |
| Lines | ✅ | ✅ | Match |

### 8.5 Projects List — WO Count Column

Di mockup Projects list, ada kolom **WORK ORDERS** yang menampilkan jumlah WO per project:

```
PR_#   PROJECT                    CUSTOMER                  PHASE           WORK ORDERS  BILLED     BALANCE
PR_07  RTU program · Bldg A + B   Northline School District  1 of 2 · Bldg A  2          $1,236.00  $2,884.00
```

Implementasi saat ini tidak punya kolom WORK ORDERS, BILLED, dan BALANCE.

---

## 9. DATABASE — WO SCHEMA vs MOCKUP

### Field yang SUDAH ADA di schema `work_orders`

| Field | Tipe | Ada di mockup | Deskripsi |
|-------|------|--------------|-----------|
| customerId | FK→customers | ✅ | Customer |
| siteId | FK→sites | ✅ | Site/location |
| status | varchar | ✅ | new/scheduled/in_progress/complete/... |
| jobTypeValueId | FK→value_sets | ✅ | Job type |
| priorityValueId | FK→value_sets | ✅ | Priority |
| assignedMemberId | FK→members | ✅ | Assigned tech |
| summary | text | ✅ | Description |
| notes | text | ✅ | Internal notes |
| schedDate/Start/End | varchar | ✅ | Schedule |
| **projectId** | text | ✅ | **Ada di DB tapi belum di form!** |
| equipmentId | text | ✅ | Equipment link |
| depositRequired | boolean | ✅ | Deposit gate |
| depositAmountCents | integer | ✅ | Deposit amount |
| depositInvoiceId | text | ✅ | Deposit invoice link |
| subtotalCents | integer | ✅ | Lines total |
| totalCents | integer | ✅ | Final total |
| paidCents | integer | ✅ | Amount paid |
| balanceCents | integer | ✅ | Remaining balance |
| createdFrom | varchar | ✅ | Source (quote/lead/project) |
| quoteId | text | ✅ | Source quote |
| agreementId | text | — | Service plan link |

### Field yang BELUM ADA tapi perlu

| Field | Tipe | Deskripsi |
|-------|------|-----------|
| **phaseId** | text FK→project_phases | Link WO ke specific phase/WBS item |
| **crewId** | text FK→crews | Crew assignment (table belum ada) |

---

## 10. ALUR DATA — BAGAIMANA SEMUA SALING TERHUBUNG

### 10.1 WO → Time → Invoice

```
Tech start timer di PWA
  ↓
time_entries terbuat (work_order_id = WO_214, billable = true, 2.5 jam)
  ↓
Admin lihat di WO → tab Time → "2.5h billable"
  ↓
Admin "Add to invoice" → invoice line:
  "Site labor · 2.5h × $85/hr = $212.50"
```

### 10.2 WO → Materials → Invoice

```
Tech tambah material line di PWA/Admin
  ↓
wo_lines terbuat (kind = "added", item = "Crane lift", $450)
  ↓
Muncul di WO → tab Materials
  ↓
Create invoice → line included
```

### 10.3 WO → Expenses → Invoice

```
Tech beli material di lapangan
  ↓
expense terbuat (work_order_id = WO_214, amount = $50, billable = true)
  ↓
Tech submit → Admin approve
  ↓
Admin "Bill expense" → invoice line: "$50 × 1.15 (15% markup) = $57.50"
```

### 10.4 WO → Project → WBS Budget

```
WO_214 linked ke Project PR_07, Phase 1.1
  ↓
WO total ($4,120) → Phase 1.1 actual naik
  ↓
Time entries cost → Phase 1.1 labor cost naik
  ↓
Expenses → Phase 1.1 actual naik
  ↓
WBS tab: Budget $6,400 vs Actual $3,140 → 40% complete
```

### 10.5 WO → Equipment Birth

```
WO line: "AC Unit" (creates_equipment = true)
  ↓
Tech complete checklist:
  - install_serial = "SN-2024-PARK-001"
  - install_nameplate_photo = uploaded
  ↓
Complete WO → server auto-creates equipment record:
  - name: "AC Unit"
  - serial: "SN-2024-PARK-001"
  - customer: sama dengan WO
  - site: sama dengan WO
  - install_wo_id: WO_214
```

---

## 11. WO STATUS LIFECYCLE

```
        ┌──────┐
        │ new  │ ← Baru dibuat, belum dijadwalkan
        └──┬───┘
           │ Admin schedule visit (tanggal + tech)
           ▼
     ┌───────────┐
     │ scheduled │ ← Visit dijadwalkan, muncul di dispatch board
     └─────┬─────┘
           │ Tech tap "Start visit" di PWA
           ▼
    ┌─────────────┐
    │ in_progress │ ← Tech sedang mengerjakan
    └──────┬──────┘
           │ Tech/Admin complete WO
           ▼
     ┌──────────┐
     │ complete │ ← Selesai, bisa buat invoice
     └────┬─────┘
          │ Admin create invoice
          ▼
     ┌──────────┐
     │ invoiced │ ← Invoice sudah terbuat
     └──────────┘

  Branching states:
     ┌──────────┐
     │ on_hold  │ ← Ditunda (menunggu parts, approval, dll)
     └──────────┘
     ┌──────────┐
     │ canceled │ ← Dibatalkan
     └──────────┘
```

---

## 12. RINGKASAN — ALUR LENGKAP

```
┌─────────────────────────────────────────────────────────────┐
│                        ADMIN                                 │
│                                                              │
│  1. Buat WO (manual / dari quote / dari project)             │
│     → Customer, Site, Job type, Tech, Lines, Schedule        │
│  2. WO muncul di dispatch board                              │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │                   TECH (PWA)                            │  │
│  │                                                         │  │
│  │  3. Lihat My Day → Start visit                          │  │
│  │  4. Kerjakan: Checklist + Photos + Time + Materials     │  │
│  │  5. Checkout: Review totals → Customer sign → Pay       │  │
│  │  6. Complete job                                        │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  7. Review: Time entries, Expenses → Approve                 │
│  8. Create invoice (WO lines + billable time + expenses)     │
│  9. Send invoice → Customer bayar → Invoice paid             │
│  10. Payout ke bank                                          │
│                                                              │
│  MONITORING TABS:                                            │
│  Overview  → Ringkasan WO                                    │
│  Visits    → Schedule kunjungan                               │
│  Time      → Jam kerja + labor cost                           │
│  Materials → Material terpakai                                │
│  Expenses  → Pengeluaran lapangan (ad-hoc)                   │
│  Photos    → Dokumentasi visual                               │
│  Payments  → History pembayaran                               │
│  Forms     → Checklist completion                             │
│  Sign-offs → Work authorization signatures                    │
│  Files     → Dokumen (PDF, permits)                           │
│  Messages  → Komunikasi customer                              │
│  Related   → Links ke Quote, Invoice, Equipment, Project     │
│  Activity  → Audit trail                                      │
└─────────────────────────────────────────────────────────────┘
```

---

## 13. PERTANYAAN UNTUK CLIENT

1. **Materials vs Expenses di PWA Tech:** Di mockup PWA tech ada "Materials" (item lines), di implementasi ada "Expenses" (pengeluaran ad-hoc dengan approval). Apakah keduanya dibutuhkan, atau Materials saja cukup?

2. **Crew assignment:** Mockup punya field "Crew" di WO. Apakah fitur crew management (grouping tech) dibutuhkan sekarang?

3. **Text customer dari WO:** Mockup punya action "Text customer…" langsung dari WO. Apakah ini prioritas?

4. **Download PDF:** Mockup punya action "Download PDF". Apakah WO perlu bisa di-export ke PDF?

5. **Project linking dari WO form:** Saat buat WO baru, apakah admin perlu bisa pilih Project dan Phase langsung dari form WO New, atau hanya bisa dari tombol "New WO" di Project Record?

6. **Sign-offs tab:** Implementasi punya tab Sign-offs terpisah (work authorization, NTE). Di mockup ini muncul di Related sebagai "Signed document". Apakah tab terpisah lebih baik (implementasi sekarang), atau cukup di Related?
