# 📊 GREE Mexico Sales Performance Report

A sales, collections, and commission reporting tool. Single-file HTML, no backend required, deployed on GitHub Pages. Roster, targets, China Projects, commission rules, and commission inputs are stored in Google Sheets.

A **left navigation** splits the app into two areas, because the two run on different cadences (sales updates daily; commission is a once-a-month calculation):

- **📊 Sales Report** — the daily sales/collections tool, containing five tabs: **Collection Report**, **Sales Report**, **Target Setup**, **China Projects**, **Upload Data**.
- **💰 Commission** — the monthly commission calculator, with three sub-tabs: **RAC**, **L&CAC**, **Inputs**.

> The Sales Report view opens on the **Upload Data** tab by default. On narrow screens the left navigation collapses to a top bar. All amounts are displayed in **MXN** (thousands separator; two decimals in the app, whole-peso totals in commission payouts).

---

## 🔗 Key Links

| Name | URL |
|------|-----|
| **Web app** | https://zoezhao273.github.io/greemexico-salesreport2/gree_sales_report2.html |
| **GitHub repository** | https://github.com/zoezhao273/greemexico-salesreport2 |

> The cloud backend (Google Sheets via an Apps Script Web App) endpoint is configured inside the HTML (`GS_URL`) and is intentionally not reproduced here. Uploading a new HTML file to the repository does not change any of these links.

---

## 📁 Files

| File | Description |
|------|-------------|
| `gree_sales_report2.html` (deployed name; local development file is `index.html`) | Complete single-file web app containing all modules |
| `README.md` | This documentation |

External libraries are loaded via CDN: SheetJS (Excel parsing), html2canvas (PNG export), and JSZip (bundling the commission export into a single `.zip`).

---

## 🧭 Module Overview

### 📊 Sales Report view (five tabs)

| Tab | Purpose |
|-----|---------|
| **Collection Report** | Monthly and latest-day collections by department |
| **Sales Report** | Salesperson target achievement by month and branch, including Other Dept and All Branches Summary; a collapsible **Exclusions** panel below the report drops specific SO Numbers from all calculations |
| **Target Setup** | Manage monthly rosters, targets, metric schemas, and per-branch **business group** (RAC-led / CAC-led); opens **read-only** with an Edit toggle |
| **China Projects** | Manual entry of domestic China deals not in the ERP/SI data; injected into both Sales Report and commission calculations |
| **Upload Data** | Upload four data sources: SI file, Customer Basic Info, Collection Doc, and **Sales Order Report** (for commission brokerage allocation) |

### 💰 Commission view (three sub-tabs)

| Sub-tab | Purpose |
|---------|---------|
| **RAC** | Residential commission — per-set salesperson rule, tiered BM rule, and the RAC results (Salesperson + BM streams) |
| **L&CAC** | Commercial commission — the RAC-led / CAC-led tier rule and the L&CAC results (Salesperson + BM streams) |
| **Inputs** | 3rd Brokerage Fee, Collection Progress, and the Commission-paid ledger |

> The month selector, **📤 Post month to Commission-paid**, **📊 Export Excel** button, and the running RAC / L&CAC / Total figures sit above the sub-tabs and apply to all three.

---

## 1️⃣ Upload Data

Four upload cards in a responsive grid (stacked on narrow screens). Each card supports click or drag-and-drop. A **✕ Clear** button appears top-right when a file is loaded (SI / Customer / Collection). On successful upload, the drop zone shows the filename and a summary. **All uploaded data is processed in browser memory only — nothing is sent to a server** (except the SO Report, whose extracted per-SO totals are saved to the cloud so historical commission months stay recomputable).

| Card | Badge | File requirement | Key fields |
|------|-------|-----------------|------------|
| **Upload SI (Sales Invoice)** | for Sales Report | `.xlsx` with **one sheet per month** (e.g. `August 2026`) | See Data Processing Rules |
| **Upload Customer Basic Info** | for L&CAC Cus | `.xlsx` with columns `Customer Code`, `Customer Name`, `RFC`, and `First Cooperate Date` | Used to calculate new L&CAC customer count (matched by Customer Code) and to run an RFC duplicate check |
| **Upload Sales Order Report** | for Commission | `.xlsx` with a **`Detail Sales Order`** sheet | `Order Code`, `Item Code`, `Final Sub Total Without Tax` — builds each SO's RAC+LAC+CAC base for brokerage-fee allocation |
| **Upload Collection Doc** | for Collection Report | `.xlsx` | `Doc Date`, `Department`, `Debit Amt in FC` |

- **SI file**: after upload the drop zone shows `{n} month(s) · {m} invoice lines · {k} orders (SO)`. SI represents invoiced, customer-signed deliveries — replacing the old Sales Order Report (order-entry basis, which included unshipped orders). A single SI file covers all months of the year; there is no need to upload files month by month. If any lines are missing a `Customer Code`, an amber note `⚠️ {n} line(s) without Customer Code` is appended — those lines cannot be matched for L&CAC new-customer development (see Data Processing Rules).
- **Customer Basic Info**: shows `Customers {n}` after upload. Customers missing a First Cooperate Date are noted in muted text. New-customer development is matched by **`Customer Code`** (see Data Processing Rules → L&CAC new customers).
  - **RFC duplicate check** ⭐: after upload, the app groups customers by `RFC` (tax ID) — each real customer must have a unique RFC. If any non-exempt RFC appears on two or more customer records, an amber warning block appears **directly below the upload box**, listing each duplicated RFC with its Customer Codes and names. This flags the same entity created more than once in the ERP (which would otherwise double-count new-customer development); clean or merge the records at the source. The check is **advisory only** — the file still loads and all calculations run normally.
  - **Exemptions**: RFCs `BDS2302216Q2` and `XAXX010101000` are allowed to repeat and never trigger a warning. **Blank** RFCs are skipped (cannot be verified). Placeholder values such as `-` are **not** exempt — if shared, they are reported. Rows without a Customer Code (e.g. export footer/junk rows) are ignored.
- **Sales Order Report** ⭐: reads the **`Detail Sales Order`** sheet, sums `Final Sub Total Without Tax` over the RAC/LAC/CAC rows (`MXR`/`MXL`/`MXC` prefixes) per `Order Code`, **rounds each to a whole peso**, and stores the result as that SO's three-line total — the denominator for brokerage-fee allocation. The upload **merges by SO (upsert), never wipes**: SOs in the file are added or updated; SOs already in the cloud but absent from the file are **kept** (so older orders never lose their denominator). After upload a summary reports `{n} added · {m} updated · {k} kept`, and **any SO whose denominator value changed is listed explicitly** (a changed denominator re-settles already-paid invoices on that SO next month). Stored under the `soDenoms` cloud key.
- **Collection Doc**: shows `{n} docs · {earliest date} → {latest date}`. Doc Date accepts `yyyy-mm-dd` text, native date cells, and Excel serial numbers.

> **SI parsing**: the app reads every sheet whose name resolves to a `yyyy-mm` month (format: English month name + year, e.g. `August 2026`) and normalises each row into a flat invoice line. It also retains, per month, the untouched source rows (used for the "SI Base" sheet in the commission export) and each line's `Item Code` and `Invoice Code` (used by the commission engine).

---

## 2️⃣ Sales Report

View salesperson target achievement by **month** and **branch**.

### Branch dropdown
Top to bottom:
1. Each rostered branch (e.g. Mexico City Branch, Monterrey Branch, Project Dept)
2. **Other Dept** — orders whose salesperson matches no roster entry (see Data Processing Rules)
3. **All Branches Summary** — side-by-side comparison of all branches

> **ERP exclude filters have been removed.** The old Exclude Deleted / Lock Stock·Inner Use / Not-Exist toggles applied to the order-entry data source (draft, stock-locked, and internal orders that had not shipped). SI data is already delivered and signed-off, so those statuses do not generate SI lines. The toggles have been removed from both the UI and all calculations.

### Single-branch view
- First row: branch total (`∑ {branch} Total`)
- Then: team leads (👑) / group subtotals + individual salespeople
- Each metric (defined by the schema) shows **Target / Progress / Achv%**
- Achievement colour: 🟢 ≥ 100%　🟡 50–99%　🔴 < 50%

### Other Dept view
Orders whose salesperson's **employee ID and name both fail to match any roster entry** are grouped here.
- Flat table: `∑ Other Dept Total` row + one row per salesperson with a **Dept** column (raw `Dept Name` from SI), grouped by department, no department subtotals.
- No targets → **Target and Achv% columns are hidden**; only Progress is shown.

### All Branches Summary view
Side-by-side comparison of all branches. Column groups per row:

| Group | Contents |
|-------|---------|
| Orders / Customers | Valid order count and unique customer count |
| RAC | Target(qty) · Qty · Amt · Achv% · Qty(prev month) · MoM% |
| LAC | Target(amt) · Qty · Amt · Achv% · Amt(prev month) · MoM% |
| CAC | Target(amt) · Qty · Amt · Achv% · Amt(prev month) · MoM% |
| Total | Combined Qty + combined Amt for the month |
| YTD | Jan-to-current-month Qty/Amt per line + combined (rostered months only) |

- The `🌐 All Branches` row is the grand total and **includes Other Dept** — orders, customers, all line Qty/Amt, prior-month MoM base, and YTD are all folded in, so the total row equals the sum of every row beneath it.
- The **Other Dept** row remains a separate line at the bottom with the same column structure; Target/Achv% show `—` (no targets); Qty/Amt/MoM/YTD are calculated normally.
- Note: because Other Dept has no targets, the **Achv%** in the grand total row uses only rostered branch targets as the denominator (while the numerator includes Other Dept progress), so achievement percentage will be slightly overstated. All additive columns (Qty, Amt, YTD) are exact sums.
- MoM% colours: ▲ green (growth) ▼ red (decline).

**Export:** every Sales Report view supports PNG and CSV export. The All Branches Summary PNG renders at full width. The CSV `ALL BRANCHES` row includes Other Dept.

### Exclusions (collapsible panel below the report) ⭐
The SI file has no order-type field, so it cannot flag orders that should not count toward sales — e.g. giveaway, inner-use, or sample orders. The **Exclusions** panel lets you drop such orders by SO Number. It sits **below the report on the Sales Report tab and is collapsed by default**; click to expand.

| Column | Type | Notes |
|--------|------|-------|
| **SO No.** | text | The only field that affects calculations. |
| **SO Type** | dropdown: `Standard` / `Inner Use` / `Sample` | Label only — does not affect exclusion. |
| **Usage** | dropdown: `Free Promotional Giveaway` / `Inner Use - Fixed Asset` | Label only. |

- **Any SO Number listed is dropped entirely** from the Sales Report — amounts, quantities, order counts, and customer counts — **regardless of SO Type**. `Standard` is also excluded (some giveaway orders are created as Standard in the ERP), so exclusion depends solely on the SO No. being present.
- Exclusion applies uniformly across single-branch views, Other Dept, and All Branches Summary (four SI aggregation loops share one exclusion set), and therefore flows into YTD, MoM, CSV, and PNG automatically.
- An SO Number that is **not in the current SI file** is simply ignored (no error). Such rows show a small `not in current SI` hint.
- Each row has a **🗑 delete** button. Clearing a row's SO No. and leaving the field auto-removes that row. Blank rows are dropped on save.
- Data is stored under the **`exclusions`** key in Google Sheets. Click **☁️ Save to Cloud** to persist. Dropdowns default to `Standard` / `Free Promotional Giveaway` on a new row.

> Note: an SO whose lines are all accessories/service (no `MXR/MXL/MXC` line) never enters the sales calculation in the first place, so listing it here is harmless but has no numeric effect. Exclusions matter for SOs that **do** contain RAC/LAC/CAC lines but should be removed for business reasons (giveaway / inner-use / sample).

---

## 3️⃣ China Projects (Manual Entry)

Records domestic China delivery deals **not present in the ERP or SI data**. Entries are automatically converted to MXN and **injected into Sales Report calculations** (`computeP`), so single-branch views, Other Dept, All Branches Summary, YTD, MoM, CSV, and PNG all include this data automatically.

- **Attribution**: by **branch + employee ID** (`empBranch` / `empId`). The dropdown stores the Employee ID, not free text. Legacy rows without `empBranch` are attributed only when the ID is unambiguous (unique across the current month's roster); otherwise the row is skipped and a re-selection is prompted — preventing cross-branch double-counting.
- **Amount**: `Sales amount = SO Amt exclude tax × FE Rate` → MXN (rows in MXN use ×1). **Quantity = Qty. Product line = Line.**
- **Unbilled rows** (exclude-tax amount is blank or 0) contribute neither amount nor quantity.
- **Counting**: each project counts as **1 order and 1 customer** (deduplicated by project name). China projects are inherently valid orders and are not affected by any filters.
- **Default mode**: read-only (MXN). A toggle at the top switches to Edit mode (original currency).
- **Shipping Progress**: the last column is a fixed 3-option status (`Not Shipped` / `Partially Shipped` / `Fully Shipped`), shared with Part 2's Shipping Progress (stored in the row's `progress` field; any legacy free-text value is preserved as a selectable option). A project that was not fully collected from the start is auto-pulled into the commission **Collection & Shipping** table, where its Collection % and Shipping Progress are editable and **sync both ways** with this module.
- Data is stored under the `chinaProjects` key in the cloud backend. Click **☁️ Save to Cloud** to persist. The module has its own **🖼 Export PNG** (off-screen full-width table).

---

## 4️⃣ Collection Report

Tracks monthly collections by department — useful for monitoring month-to-date totals and the most recent day's receipts.

- Select a **month** (`yyyy-mm`) from months present in the uploaded Collection Doc.
- The table shows two columns side by side:
  - **Month-to-Date**: cumulative collections for the selected month
  - **Latest-Date**: collections on the **last Doc Date** within the selected month (for the current month this equals today; for past months it is the last day of that month — no cross-month leakage)
- Each column has a `∑ Total` row plus one row per department.
- Amounts displayed in **MXN**.
- **Export**: PNG and CSV available top-right.

### Department rules
- **Finance Dept** is displayed as **Pending**.
- Sort order: departments listed in Target Setup (roster order) → other departments (alphabetical) → Pending last.

> The Collection Doc parser auto-detects the header row within the first 20 rows (some ERP exports include a banner row before the real header) and uses column-name aliases (`DocDate`/`Doc Date`, `ResponsibleDept`/`Department`, `AmtinFC`/`Debit Amt in FC`), remaining backward-compatible with older export formats.

---

## 5️⃣ Target Setup (Monthly Roster + Targets + Metric Schema)

**Rosters, targets, and metric schemas are stored independently per month.**

- **Opens read-only** ⭐: Target Setup opens locked every time. Click **✏️ Edit** (top-right) to unlock — this reveals **☁️ Save to Cloud** and **🏢 Add branch** and enables all inputs, drag handles, and delete buttons. Click **🔒 Done** to lock again. Creating a roster (Copy / Start blank) auto-unlocks.
- Select a year and month. New months can be initialised by **📋 Copy from previous month** or **✨ Start blank** (copies only from the immediately preceding month — no chain search).
- The first row of each department is a read-only auto-sum total. 👑 marks team leads (optional targets). Row order and column widths are drag-adjustable.
- **Business group** ⭐: each branch has a **RAC-led / CAC-led** selector next to its name (defaults to RAC-led), stored per month. It picks the commercial-commission rate column for that branch (see Commission → L&CAC) and **does not affect the residential BM logic**. Stored inside the `rosters` key as each branch's `bizGroup` field.
- **Employee IDs must be unique and must match the `Salesman Code` (employee number) in the SI file** ⭐:
  - Attribution now relies entirely on the employee ID. If a roster ID does not match the SI Salesman Code, that person's orders will fall into Other Dept.
  - Duplicate IDs within the same branch cause data to be merged incorrectly; a validation check runs before saving.
- **Metric schema** controls which columns appear in the Sales Report. Each metric is `{key, label, unit, src}`:
  - `unit`: `qty` (quantity) or `amt` (amount excl. tax)
  - `src`: `RAC` / `LAC` / `CAC` / `LAC+CAC` (combined L&CAC amount) / `CUST` (new L&CAC customers, matched by Customer Code — requires Customer Basic Info upload)
  - Default schema: RAC (qty), LAC (amt), CAC (amt)
- Click **☁️ Save to Cloud** to save roster, targets, and schema.

---

## 🏗️ Data Processing Rules (Sales Report)

> From August 2026 the data source changed from **Sales Order Report (order-entry basis)** to **SI — Sales Invoice (delivered and signed-off basis)**. The rules below reflect the current live configuration.

### Data structure
SI is a **single flat file with one sheet per month**: each row is one product line within one invoice. Header fields (invoice, department, salesperson, customer) repeat on every row — no header-to-detail join is required. Each invoice and each SO maps to exactly one department, salesperson, and customer.

### Salesperson attribution
Attribution follows the **roster**, not the `Dept Name` field in SI. Match priority:
1. **Exact employee ID match**: SI `Salesman Code` equals roster `id` → attributed immediately.
2. **Name fuzzy fallback** (when ID does not match): exact name → substring match → keyword overlap (words longer than 2 chars, ≥ 2 overlapping words required).

> Prerequisite: roster `id` must equal the employee's Salesman Code. Verify this when setting up or updating the roster.

### Product line classification ⭐
Determined by **Item Code prefix** (Business Unit Code is ignored):

| Prefix | Line | Metrics |
|--------|------|---------|
| `MXR…` | **RAC** | Qty + Amt (excl. tax) |
| `MXL…` | **LAC** | Amt (excl. tax) + Qty |
| `MXC…` | **CAC** | Amt (excl. tax) + Qty |
| Anything else | **Not counted** | — |

"Anything else" includes: `MXI` (not CAC), long numeric codes (spare parts — main boards, motors, compressors, remotes), `SRVS` service, `ASC` freight, etc.

> Using the Item Code prefix rather than Business Unit Code automatically strips accessories and service lines that are tagged under a product-line BU, and rescues genuine units whose Business Unit Code was left blank.

### Valid order rule ⭐
**Order unit = `SO Number`** (one SO may span multiple invoices and/or months). An SO is counted toward **Orders and Customers only if it contains at least one RAC/LAC/CAC line**. Accessory-only, service-only, or freight-only SOs are excluded from order and customer counts (they contribute zero to all product-line amounts anyway). This rule applies to single-branch views, Other Dept, and All Branches Summary.

### Amount and quantity
- Amount: **`Sub Amount After Discount Without Tax`** (after discount, before down-payment, excl. tax)
- Quantity: **`Quantity`**

### Cross-month SO handling (Plan B) ⭐
When a `SO Number` has invoices spread across multiple month sheets:
- **Amount / Quantity**: counted in each invoice's **own month** (each month counts its own lines).
- **Order count / Customer count**: counted only in the SO's **earliest month** — never double-counted in later months.

> No cross-month SOs exist in the current 8-month file; this rule is a forward-looking safeguard.

### Other Dept attribution
When a salesperson's **employee ID and name both fail to match any roster entry**, their orders are placed in Other Dept, grouped by raw `Dept Name`, then by salesperson. Only valid orders are included. No targets.

### L&CAC new customers (the `CUST` metric) ⭐
Counts customers **newly developed in the selected month** who placed an L&CAC order — i.e. the count of distinct customers whose **First Cooperate Date falls in the report month** and who have at least one valid SO containing an `MXL`/`MXC` (LAC/CAC) line that month. It is a *new-customer* count, not a count of all customers with L&CAC activity.

- **Matched by `Customer Code` (exact, strict)** ⭐: the SI `Customer Code` (column F) is matched against the Customer Basic Info `Customer Code` (column B). Both files come from the same source, so codes are compared **as-is** (no case/space normalisation). Matching by code — not by customer name — avoids the errors that arise when one name maps to several codes, or when a name is spelled differently across files.
- **Strict mode**: if an SI line has **no** Customer Code it is **not** counted as a new customer (there is no name-based fallback). The SI upload reports how many lines lack a code so this is transparent.
- **Deduplication**: counted as a `Set` of Customer Codes, so one new customer with several L&CAC orders counts once. Branch/company subtotals union the per-salesperson code sets, so the same new customer credited to two salespeople still counts once for the team.
- **Prerequisite**: the Customer Basic Info file must be uploaded. Without it, the `CUST` column shows `⤒ file` and contributes nothing to any total.

> A repeated RFC in Customer Basic Info (same entity, two Customer Codes) would let one real customer be counted as new twice if both codes trade under L&CAC. The RFC duplicate check on upload surfaces exactly these cases so they can be cleaned at the source.

### YTD calculation
- Accumulated from January of the current year through the selected month.
- Each month uses **its own roster** for attribution. If a month has no roster, the **previous month's roster** is used. If neither exists, that month is excluded from YTD.

### Month identification
- Sales: **sheet name** takes precedence (`August 2026` → `2026-08`). `Invoice Date` (`yyyy-mm-dd`) is used for display/validation only.
- Collections: `Doc Date` (`yyyy-mm-dd`, plus native date cells and Excel serial numbers).

---

## 💰 Commission (Monthly)

A separate top-level view that computes monthly commission from the **same SI data** as the Sales Report, plus the Sales Order Report (brokerage denominators) and three manual input tables. It reuses the roster, targets, schema, Exclusions, and `computeP` logic, so achievement rates and amounts match the Sales Report exactly (Exclusions applied, China Projects included).

> **Access control**: commission is a distinct view so it can eventually be split to a separate file — branch managers who hold the Sales Report password should not necessarily see individual pay data.

### Core model ⭐
- **Unit of calculation = one SI invoice.** A single SO can be invoiced across several months; each invoice is handled on its own.
- **Tier rate is locked to the invoice's own month.** A top-up in a later month still uses the original month's achievement tier. The first time an invoice is posted, its **100%-collection commission (`Full`)** is frozen into the Commission-paid ledger; from then on `Full` is the authoritative base and is reused unchanged in later months.
- **Incremental payout against collection progress**: `payout this month = Full × current collection% − amount already paid in earlier months` (from the Commission-paid ledger). As collection rises, the balance is topped up; once fully collected and fully paid, the invoice disappears from the tables.
- **Cross-month top-ups do not require the SI file to keep history ⭐**: the engine reads from two sources and de-duplicates by `(Type, SI)` — invoices still in the uploaded file use the file path (with the frozen `Full` applied), and any posted `(Type, SI)` **no longer present in the file** is paid straight from the ledger (frozen `Full` × current collection% − already-paid). So once a month is posted, later top-ups are correct even if a subsequent SI upload omits earlier months. Owner/branch for a vanished-invoice top-up come from the frozen ledger row.
- **Rounding**: `Full` and `SI Base` are stored at full real precision (float-noise stripped, not truncated to cents); each month's payout is stored to the cent; per-person monthly display totals are rounded to whole pesos.

### Four commission streams
A single invoice can generate up to four independent streams, each tracked and paid separately:

| Stream | Who | Line | Achievement basis | Rate source |
|--------|-----|------|-------------------|-------------|
| **RAC-SP** | Salesperson | RAC | none (per-set) | per-set price table |
| **RAC-BM** | BM (branch/group lead) | RAC | unit RAC qty ÷ target | RAC BM tier table |
| **L&CAC-SP** | Salesperson | MXL+MXC | **personal** amount ÷ personal target | L&CAC tier table |
| **L&CAC-BM** | BM (branch/group lead) | MXL+MXC | **unit** amount ÷ unit target | L&CAC tier table |

A person who is an active BM of their unit is excluded from the corresponding salesperson stream. The Commission-paid ledger carries a **Type** column to keep the four streams separate (essential for correct top-ups); a special `Paid` type marks a whole-SI settlement (see Part 4 below).

### Formulas
**Brokerage-fee allocation** (RAC-BM / L&CAC-SP / L&CAC-BM only):
`allocated fee = SO brokerage fee × (invoice product-line amount ÷ SO three-line total)`
The three-line total (RAC + LAC + CAC) comes from the uploaded Sales Order Report (`soDenoms`). RAC streams use the invoice RAC amount as numerator; L&CAC streams use the invoice MXL+MXC amount. **RAC-SP does not deduct any brokerage fee.**

**Calculation granularity = one SI invoice.** All four streams compute per-invoice; results are aggregated to a per-person total at the end.

---

#### A · RAC-SP (Residential · Salesperson)
1. **Gross** = Σ (item qty × per-set rate) — from `commRuleSP`
2. **Full** = Gross *(no brokerage deduction)*
3. **Collection %** = `commCollection[SO].pct` (default 100 % if not entered)
4. **Paid-before** = Σ `commPaid` rows where `type = RAC-SP`, `SI = this invoice`, `month < M`
5. **This-invoice pay** = Full × Collection% − Paid-before (floored to 0 if negative)
6. **Person total** = Σ this-invoice pay across all invoices → **rounded to whole pesos**

#### B · RAC-BM (Residential · Branch Manager)
1. **Allocated fee** = SO brokerage fee × (invoice RAC amount ÷ SO three-line total)
2. **Net base** = invoice RAC amount excl. tax − allocated fee
3. **Tier rate** = look up branch RAC achievement rate in `commRuleBM` — **locked to the invoice's own month**
4. **Full** = Net base × Tier rate
5. **Collection %** = `commCollection[SO].pct` (default 100 %)
6. **Paid-before** = Σ `commPaid` rows where `type = RAC-BM`, `SI = this invoice`, `month < M`
7. **This-invoice pay** = Full × Collection% − Paid-before (floored to 0 if negative)
8. **Person total** = Σ this-invoice pay across **all invoices belonging to the BM's unit** → **rounded to whole pesos** *(critical: the BM's payout is the rounded sum of all per-invoice pays, not a per-invoice rounded figure)*

#### C · L&CAC-SP (Commercial · Salesperson)
1. **Allocated fee** = SO brokerage fee × (invoice MXL+MXC amount ÷ SO three-line total)
2. **Net base** = invoice MXL+MXC amount excl. tax − allocated fee
3. **Tier rate** = look up **personal** L&CAC achievement rate in `commRuleCom` (rate column by branch business group) — **locked to the invoice's own month**
4. **Full** = Net base × Tier rate
5. **Collection %** = `commCollection[SO].pct` (default 100 %)
6. **Paid-before** = Σ `commPaid` rows where `type = COM-SP`, `SI = this invoice`, `month < M`
7. **This-invoice pay** = Full × Collection% − Paid-before (floored to 0 if negative)
8. **Person total** = Σ this-invoice pay across all invoices → **rounded to whole pesos**

#### D · L&CAC-BM (Commercial · Branch Manager)
1. **Allocated fee** = SO brokerage fee × (invoice MXL+MXC amount ÷ SO three-line total)
2. **Net base** = invoice MXL+MXC amount excl. tax − allocated fee
3. **Tier rate** = look up **unit (branch/group)** L&CAC achievement rate in `commRuleCom` — **locked to the invoice's own month**
4. **Full** = Net base × Tier rate
5. **Collection %** = `commCollection[SO].pct` (default 100 %)
6. **Paid-before** = Σ `commPaid` rows where `type = COM-BM`, `SI = this invoice`, `month < M`
7. **This-invoice pay** = Full × Collection% − Paid-before (floored to 0 if negative)
8. **Person total** = Σ this-invoice pay across **all invoices belonging to the BM's unit** → **rounded to whole pesos** *(critical: same aggregation rule as RAC-BM)*

---

| | Brokerage deducted | Achievement basis | Rate table |
|---|---|---|---|
| **RAC-SP** | ❌ No | Fixed per-set (no tier) | `commRuleSP` |
| **RAC-BM** | ✅ Yes | Branch RAC achievement rate | `commRuleBM` |
| **L&CAC-SP** | ✅ Yes | Personal L&CAC achievement rate | `commRuleCom` |
| **L&CAC-BM** | ✅ Yes | Unit L&CAC achievement rate | `commRuleCom` |

### "Unit" = branch, or group when a branch has groups ⭐
When a branch has any grouped person (the Target Setup **Group** field), **each group behaves as its own department**: the group's 👑 lead is that group's BM, achievement is measured over the group's members only, and only that group's invoices feed its BM stream. A branch with no groups uses whole-branch scope. This applies to both RAC-BM and L&CAC-BM. (Mixed grouped/ungrouped within one branch does not occur.)

### Module 1 · RAC (Residential)
Two rule tables (collapsible, **collapsed by default**, read-only with an Edit → Save-to-cloud toggle):
- **RAC Commission Rule — Salesperson** (`commRuleSP`): per Item Code, `Commission/set (MXN)`. 35 default items.
- **RAC Commission Rule — BM** (`commRuleBM`): achievement tiers → BM commission rate (`<60%` = 0). Applied to the unit's RAC net sales excl. tax.

The RAC results card has two sections: **Salesperson (per-set)** and **BM (tier × unit RAC net)**.

### Module 2 · L&CAC (Commercial)
- **L&CAC Commission Rule — RAC-led / CAC-led** (`commRuleCom`): achievement tiers, two rate columns. The column is chosen by the branch's **business group** in the invoice month. Collapsed by default; read-only with Edit toggle. Shows the current month's RAC-led / CAC-led branch split for reference.
- Results card has **Salesperson (personal tier)** and **BM (unit tier)** sections.

### Module 3 · Inputs (`commBrokerage` / `commCollection` / `commPaid`)
One card, read-only with an Edit toggle. In read-only mode the **SO rows within each of Parts 1-3 can be drag-reordered** by the ⠿ handle at the start of each row — the reordered array is saved to that section's own cloud key (`commBrokerage` / `commCollection` / `commExempt`).

**Per-column filters (read-only view).** Every column header in all four Parts carries a ▾ funnel button. Clicking it opens an Excel-style checkbox list of that column's distinct values (with a search box + Select all / Clear); Apply keeps only the matching rows. Filters combine with **AND** across columns; a green banner shows *"showing X of Y"* with a **Clear filters** button, and the section count switches to `X / Y`. Filters are a pure **viewing aid** — they are not persisted, not shown in Edit mode, and never alter the data: rows are always rendered at their true array index and merely hidden with `display:none`, so drag-reorder indices and Edit-mode value collection are unaffected. Sections:
1. **3rd Brokerage Fee** (expanded): `SO No.` → `3rd Brokerage fee (MXN, Excl. IVA)`. SOs not listed → fee 0.
2. **Collection & Shipping** (expanded): `SO No.` → `Collection Progress (%)` → `Shipping Progress`. SOs not listed → 100% collection. Enter cumulative collection as of month-end. **Shipping Progress** is a fixed 3-option status (`Not Shipped` / `Partially Shipped` / `Fully Shipped`) — a label only; it does **not** affect any calculation. In the read-only working view, SOs at **100%** are moved into a collapsed **"✓ Fully collected · 100%"** group so the main list shows only still-open collections (purely visual — every row stays at its true array index, so the calculation and cross-month top-ups are unchanged). **China Projects** that were *not* fully collected from the start are auto-pulled into Part 2 and tagged `China Project`: their `Collection %` and `Shipping Progress` are editable here and **sync both ways** with the China Projects module (same `cloudChinaProjects` source, same shared `progress` field); at 100% they join the folded group. Projects that have always been 100% are never shown.
3. **Exemption** (expanded) ⭐: `SO No.` → `Exemption Reason`. Listed SOs are dropped from **all** commission calculations (every stream, salesperson & BM) — e.g. loss-making orders with no margin to pay commission from. Applies to SI invoices and to China projects sharing the SO No. Stored under `commExempt`.
4. **Commission-paid** (collapsed): the payout ledger — `Month`, `SO No.`, `SI No.`, `Type`, `SI Base`, `Full`, `Commission (MXN)`, `Paid%`, `Salesperson Name`, `Note`. Normally written by the Post button; editable for manual adjustments. Each posted row freezes **`SI Base`** (the SI's excl-tax amount for that stream's line), **`Full`** (the 100%-collection commission — the load-bearing base for later top-ups), the month's **`Commission`** payout and **`Paid%`** (this month's incremental percentage), plus the owner's employee ID/branch (used to re-attribute a top-up after the SI leaves the file). A month's calculation only treats **earlier-month** ledger rows as already-paid, so posting the current month does not change its own figures. Rows with **Type = `Paid`** are whole-SI settlements (paid in full, 100%): excluded from **all** streams (payout skipped, achievement unaffected), Type locked, never touched by Post/Unpost — set via the seed of pre-July 2026 paid-out SIs or manually via the Type dropdown → `Paid (settle · exclude)`.

### China Projects in commission ⭐
Each China row joins as a virtual invoice with the row's own inputs: `Project` = both the SO No. and SI No. (ledger key); `3rd Commission` = brokerage fee; `Coll. Progress` = collection %; `SO Amt excl tax` = amount (× FE Rate → MXN). It feeds the RAC-BM / L&CAC-SP / L&CAC-BM streams (per the row's product line), scoped to the owner's unit.

### Detail columns
Each detail row = one invoice × one stream: **Inv.Month · SI**, **SO**, **SI Base**, **Fee alloc**, **Achv% → Rate**, **Com Full**, **Coll%**, **Paid before**, **This month**, and a flags column. **SI Base** is always the SI's own excl-tax amount for that stream's line (RAC amount for RAC streams, MXL+MXC for L&CAC) — including RAC-SP, where it is the invoice RAC amount rather than the per-set gross.

### Flags (floor-to-zero policy) ⭐
Negative results are **never clawed back** — they floor to 0 and are flagged in amber:
- **⚠ fee &gt; commission $X**: allocated brokerage exceeds gross commission (Com Full is negative).
- **⚠ over-paid $X**: collection dropped below the already-paid ratio (returns/chargebacks).
- **PENDING …**: no target set (or salesperson not in roster) → tier undecidable, shown but not paid.
- **⚠ shared branch**: the unit has more than one 👑 lead, so each lead receives a full unit commission — verify the roster.
- **China Proj**: informational.
- **↩ top-up**: a cross-month top-up paid from the frozen ledger because the SI is no longer in the uploaded SI file (the Part 4-only path).

### 📤 Post month to Commission-paid
Computes the selected month and writes each stream's this-month payout into the ledger (keyed by Type + SI No.), freezing `SI Base`, `Full` (100%-collection commission), `Paid%`, and the owner's ID/branch on every row. Re-posting a month **replaces** that month's payout rows (a confirmation warns if manual edits for that month would be lost); `Paid` settlement rows are never touched. **Post each month before computing the next** — the next month reads earlier months' ledger rows to know what is already paid, so an un-posted month causes its invoices to be re-paid in full the following month. The write targets only that month's `commPaid_YYYY-MM` shard and is **read back to confirm** before reporting `Ledger saved & verified ✓`.

### ↩︎ Unpost this month
Reverses a Post by deleting the selected month's rows from the ledger. The button only appears when the selected month actually has ledger rows. Because the ledger is cumulative — later months are paid as **top-ups against what earlier months already paid** — if any **later** month is already posted, the confirmation is a strong warning listing those months and advising you to re-post them afterwards (otherwise their incremental amounts will be wrong). With no later months posted, it is a plain confirm. The change is saved to that month's `commPaid_YYYY-MM` shard (written, then read back to confirm). Re-posting the same month achieves the same reset without deleting first.

### 📊 Export Excel (all + per-dept)
Produces **one `.zip`** containing **1 + (n + 1)** workbooks *and* the same number of matching PNGs:
- **1** grand-total workbook (`…_ALL.xlsx`) covering everyone.
- **n** per-department workbooks (one per Target Setup branch).
- **+1** `…_Other_Dept.xlsx` for salespeople not in the roster (only when such rows exist).

Each workbook has three sheets: **Summary** (per person: code, name, RAC (ref), L&CAC (ref), and the whole-peso **Total Commission** — the Total column is the authoritative payout; the component columns are unrounded references), **Detail** (one row per invoice × stream), and **SI Base** (every raw invoice row of the month, including non-commission lines). Departments with groups are partitioned into ◆ group sections with subtotals inside the Summary/Detail sheets. Each workbook is paired with a **PNG image of its Summary** (same 1 + (n + 1) count), rendered off-screen via html2canvas. Attribution is by **roster**, not the SI `Dept Name`.

---

## ☁️ Cloud Sync

### Storage architecture

| Data | Location | Key |
|------|----------|-----|
| Monthly rosters (incl. per-branch `bizGroup`) | Google Sheets | `rosters` |
| Monthly targets | Google Sheets | `targets` |
| Monthly metric schemas | Google Sheets | `schemas` |
| China Projects | Google Sheets | `chinaProjects` |
| Exclusions (SO Numbers dropped from Sales Report) | Google Sheets | `exclusions` |
| RAC salesperson per-set rule | Google Sheets | `commRuleSP` |
| RAC BM tier rule | Google Sheets | `commRuleBM` |
| L&CAC (RAC-led/CAC-led) tier rule | Google Sheets | `commRuleCom` |
| SO three-line totals (brokerage denominators) | Google Sheets | `soDenoms` |
| 3rd Brokerage fees | Google Sheets | `commBrokerage` |
| Collection progress | Google Sheets | `commCollection` |
| Commission exemptions (SOs excluded from all commission) | Google Sheets | `commExempt` |
| Commission-paid ledger (payout + `Paid` settlement rows) | Google Sheets | **`commPaid_YYYY-MM`** (one cell per month) — legacy single-key `commPaid` is auto-migrated on first load and kept read-only as a fallback |
| Column-width preferences | Browser localStorage | Local only, not synced |
| SI / Customer / Collection Excel files | Not stored | Processed in memory on each upload |

> **New cloud keys since the commission module** — `commRuleSP`, `commRuleBM`, `commRuleCom`, `soDenoms`, `commBrokerage`, `commCollection`, `commExempt`, `commPaid`. The Apps Script store below is generic (any key), so **no backend change is needed** unless the deployment enforces a key allow-list — in which case add these keys and re-deploy.

> Legacy `roster` key compatibility: if the sheet has the old single-key `roster` but no `rosters`, it is automatically mapped to the current month on load.

> **Commission-paid ledger is sharded per month** ⭐: it is stored as one key per month (`commPaid_2026-07`, `commPaid_2026-08`, …), not a single `commPaid` cell, because Google Sheets caps a single cell at **50,000 characters** — the whole-ledger-in-one-cell approach silently overflowed once enough months were posted (see Troubleshooting). On load the app **merges every `commPaid_YYYY-MM` cell** into one in-memory ledger; if it finds only the old single `commPaid` cell it **migrates** it into per-month cells automatically (leaving the old cell intact as a read-only fallback for any un-refreshed tab). Every ledger write is **read back from the cloud and verified** before it reports success.

### Google Sheets structure
Data is stored in a sheet named `targets`. Each row is one key-value pair; the value is a JSON string:

| Column A (key) | Column B (value) |
|----------------|-----------------|
| `rosters` | `{"2026-07":[{branch, people:[{id, name, isManager, group}]}], ...}` |
| `targets` | `{"2026-07":{"Mexico City Branch":{"MX250024":{"RAC":100,"LAC":200000,"CAC":2}}}, ...}` |
| `schemas` | `{"2026-07":[{"key":"RAC","label":"RAC","unit":"qty","src":"RAC"}, ...], ...}` |
| `chinaProjects` | `[{month, feRate, empId, empBranch, project, currency, line, qty, soIncl, collProg, soExcl, thirdComm, note, progress}, ...]` |
| `commRuleSP` | `[{code, name, rate}, ...]` (per-set MXN by Item Code) |
| `commRuleBM` | `[{lo, hi, rate}, ...]` (achievement tier → BM rate; `hi:null` = ∞) |
| `commRuleCom` | `[{lo, hi, rac, cac}, ...]` (tier → RAC-led / CAC-led rate) |
| `soDenoms` | `{"MTY-20260707-002": 131393, ...}` (SO → integer RAC+LAC+CAC total) |
| `commBrokerage` | `[{so, fee}, ...]` (MXN, excl. IVA; missing SO → 0) |
| `commCollection` | `[{so, pct, ship}, ...]` (pct 0–100, missing SO → 100; ship = status label only) |
| `commExempt` | `[{so, reason}, ...]` (SOs excluded from all commission) |
| `commPaid_YYYY-MM` (one cell per month) | that month's ledger rows: `[{month, so, si, type, siBase, full, amount, paidPct, name, note, _empId, _branch}, ...]` — `type` ∈ RAC-SP / RAC-BM / COM-SP / COM-BM (payout rows) or `Paid` (whole-SI settlement, excluded from all streams). `full`/`siBase` kept at full precision; `amount` to the cent. Sharded per month so no single cell approaches the 50,000-char Sheets limit; the legacy single-key `commPaid` cell is kept read-only after migration |
| `exclusions` | `[{so, type, usage}, ...]` (only `so` affects calculations; `type` / `usage` are labels) |

### Read / write flow
- **Read (page load):** GET request to the cloud Web App endpoint (`GS_URL`, configured in the HTML); parses `rosters`, `targets`, `schemas`, `chinaProjects`, `exclusions`, and the commission keys (`commRuleSP`, `commRuleBM`, `commRuleCom`, `soDenoms`, `commBrokerage`, `commCollection`, `commExempt`); the Commission-paid ledger is assembled by **merging all `commPaid_YYYY-MM` shards** into one in-memory array, with one-time migration from the legacy single `commPaid` cell.
- **Write (Save to Cloud):** POST each key in sequence using `no-cors` mode to avoid CORS preflight issues and URL length limits. Each area saves its own keys (Target Setup → roster/targets/schemas; commission rule cards → their rule key; Module 3 / Post → the input and ledger keys; SO upload → `soDenoms`). Because `no-cors` returns an opaque response the client cannot read, the **Commission-paid ledger is verified after writing**: only the changed month-shards are POSTed, then the ledger is read back and each written shard is compared before success is reported (`Ledger saved & verified ✓`); a mismatch shows an explicit warning instead of a false confirmation.

### Apps Script code (generic key-value store)
```javascript
const SHEET_NAME = 'targets';

function doGet(e) {
  return ContentService
    .createTextOutput(JSON.stringify({ ok: true, data: getAllData() }))
    .setMimeType(ContentService.MimeType.JSON);
}
function doPost(e) {
  let result;
  try {
    const { key, value } = JSON.parse(e.postData.contents);
    setData(key, value);
    result = { ok: true };
  } catch (err) { result = { ok: false, error: err.message }; }
  return ContentService
    .createTextOutput(JSON.stringify(result))
    .setMimeType(ContentService.MimeType.JSON);
}
function getAllData() {
  const sheet = getOrCreateSheet();
  const data = sheet.getDataRange().getValues();
  const result = {};
  data.forEach(row => {
    if (row[0]) { try { result[row[0]] = JSON.parse(row[1]); } catch(e) { result[row[0]] = row[1]; } }
  });
  return result;
}
function setData(key, value) {
  const sheet = getOrCreateSheet();
  const data = sheet.getDataRange().getValues();
  for (let i = 0; i < data.length; i++) {
    if (data[i][0] === key) { sheet.getRange(i + 1, 2).setValue(JSON.stringify(value)); return; }
  }
  sheet.appendRow([key, JSON.stringify(value)]);
}
function getOrCreateSheet() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  let sheet = ss.getSheetByName(SHEET_NAME);
  if (!sheet) sheet = ss.insertSheet(SHEET_NAME);
  return sheet;
}
```

### Re-deploying Apps Script
After any code change: open the project at script.google.com → Save (Cmd+S) → Deploy → Manage deployments → pencil icon → set version to "New version" → Deploy. **The deployment URL does not change; `GS_URL` in the HTML does not need to be updated.** ⚠️ If a change adds a **new permission/scope** (e.g. `LockService`), the deployed “Anyone” web app can start returning an auth error to the app until you re-authorise and publish a new version — test the live `GS_URL` in a browser after deploying. Prefer keeping the backend generic and doing data maintenance client-side.

### Troubleshooting
| Symptom | Likely cause / fix |
|---------|-------------------|
| Save failed / Failed to fetch | Apps Script not re-deployed after a code change, or network/auth issue. Open `GS_URL` directly in a browser and check for a JSON response. |
| Cloud unavailable — offline mode | The app still works; data is in local memory only. Reload after restoring network access. |
| Changes not taking effect | Browser or CDN has cached the old HTML. Hard-refresh with `Cmd+Shift+R`. |
| Post says “saved” but the month is missing after reload | Historical bug (pre-2026-08-21): the whole ledger lived in one cell and silently overflowed the 50,000-char limit; `no-cors` + `doPost`'s try/catch hid the failure. Fixed by per-month shards + read-back verification. If you now see **⚠ Save NOT confirmed (month)**, the read-back failed — reload to check the real state and re-Post; do **not** trust that save. |
| Cloud unavailable — offline mode right after editing the Apps Script | A script edit that adds a new OAuth scope (e.g. `LockService`) breaks the live “Anyone” deployment until you re-authorise + re-deploy. Revert the change, or re-authorise and publish a new version. **Do one-time ledger maintenance from the browser, not by editing the deployed script.** |

---

## 📤 Export Reference

| Module | PNG | CSV / Excel |
|--------|-----|-------------|
| Sales Report — single branch / Other Dept | ✅ | ✅ CSV |
| All Branches Summary | ✅ (full width) | ✅ CSV (ALL BRANCHES row includes Other Dept) |
| China Projects | ✅ (off-screen full-width table) | — |
| Collection Report | ✅ | ✅ CSV |
| **Commission** | ✅ (1 + (n+1) Summary PNGs, in the zip) | ✅ **Excel** — one `.zip` of 1 + (n+1) three-sheet workbooks |

- **Export All PNG** (Sales Report): batch-exports all rostered branches + Other Dept + All Branches Summary for the selected month. A floating progress overlay survives report DOM rebuilds during the batch loop.
- **Export Excel** (Commission): see Commission → Export Excel. Bundles workbooks and Summary PNGs into a single `Commission_{month}.zip` via JSZip.
- Wide-table PNG export temporarily removes `overflow-x` constraints and expands the container before calling html2canvas, then restores styles in `.then()` and `.catch()`.

---

## 🔄 Update Procedures

### Deploying a new HTML version
1. Download the updated HTML from the development conversation.
2. GitHub repository → Add file → Upload files → overwrite the existing file (deploy name: `gree_sales_report2.html`) → Commit changes.
3. GitHub Pages updates automatically within ~1 minute. **The URL does not change.**

### Routine operations
- **Update targets:** Target Setup → select month → (for a new month: Copy from previous month) → fill in targets → Save to Cloud.
- **Run a report:** Upload Data → upload the **SI file** (+ Customer Basic Info for L&CAC new customers; + Collection Doc for the collection report) → navigate to the relevant tab → select month → view or export.
- **Log a China project:** China Projects → Add project → fill in employee ID, project name, amount, product line → Save to Cloud. The entry automatically appears in Sales Report totals.
- **Exclude an order from sales:** Sales Report → expand **Exclusions** (below the report) → Add exclusion → enter the SO No. (set SO Type / Usage labels if useful) → Save to Cloud. The SO is dropped from all Sales Report calculations.
- **Run monthly commission:** Upload Data → upload the **SI file** and **Sales Order Report** → switch to **💰 Commission** → select the month. Fill **Inputs** (Edit → 3rd Brokerage Fee, Collection Progress) → Save to Cloud. Review the RAC and L&CAC tabs → **📤 Post month to Commission-paid** → **📊 Export Excel** for the per-department zip. Commission rules only need editing when they change.

---

## 🛠️ Technical Notes

- Pure front-end single-file HTML. All logic runs in the browser; works via `file://` and GitHub Pages alike.
- External dependencies (all CDN): SheetJS (Excel parsing), html2canvas (PNG export), and JSZip (commission export bundling).
- Cloud layer: Google Apps Script (POST write / GET read) + Google Sheets.
- Hosting: GitHub Pages (public repository, free permanent URL).
- Amount formatting: `toLocaleString('es-MX')`, two decimal places (MXN).
- **Cloud cell limit:** Google Sheets caps a cell at **50,000 characters**. Large, growing datasets (the Commission-paid ledger) are **sharded by key** (one `commPaid_YYYY-MM` cell per month), never stored as a single JSON blob.
- **`no-cors` writes cannot be read back inline**, and the Apps Script `doPost` returns HTTP 200 even on an internal error — together these can hide a failed save, so the ledger **reads back and verifies every write**; other keys stay fire-and-forget.
- **Do not add scope-requiring calls to the deployed Apps Script casually** (e.g. `LockService`): a scope change breaks the live deployment until re-authorised/re-deployed. Write concurrency is handled client-side (serialised writes) instead.

---

## 📝 Changelog

| Date | Changes |
|------|---------|
| 2026-05-13 | Initial release; Google Sheets cloud storage integrated |
| 2026-05-31 | Roster/targets changed to per-month storage; Branch Summary added (MoM, YTD); cloud writes switched to POST; Customers column added; PNG export truncation fixed |
| 2026-07 | Branch Summary merged into Report branch dropdown as All Branches Summary; three exclude toggles defaulted on, Lock Stock and Inner Use merged; valid order rule added (must contain RAC/LAC/CAC line); Other Dept dimension added (dropdown entry + Summary last row); Collection Report module added (MTD + Latest-Date, Finance→Pending, dept sort); upload area redesigned to three-card compact layout with Clear buttons and in-card status; Report renamed to Sales Report; all amounts unified as MXN |
| 2026-07 (subsequent) | **China Projects** manual-entry module added (branch+ID attribution, SO Amt × FE Rate → MXN, unbilled rows skipped, 1 order + 1 customer per project, injected into computeP); fixed manager placeholder ID cross-branch data injection (composite branch+ID key); Collection Doc parser upgraded to auto-detect header row + column name aliases; Export All PNG batch export added |
| **2026-08** | **Sales data source switched from Sales Order Report (order-entry) to SI — Sales Invoice (delivered and signed-off)**. Single multi-sheet file; month = sheet name. **Product line now determined by Item Code prefix** (`MXR→RAC` / `MXL→LAC` / `MXC→CAC`; all others including MXI not counted). **Order unit changed to SO Number** (multi-invoice SO deduplication). **Salesperson attribution switched to employee ID** (Salesman Code exact match; name fuzzy fallback). **Three ERP exclude filters removed**. Cross-month SOs use **Plan B** (amounts by invoice month; order/customer counted once in earliest month). Amount column: `Sub Amount After Discount Without Tax`. **All Branches Summary grand total now includes Other Dept** (table and CSV), making the total row equal the sum of all rows beneath it |
| **2026-08 (subsequent)** | **Exclusions panel** added below the Sales Report (collapsed by default) — lists SO Numbers to drop entirely from all sales calculations regardless of SO Type; stored under the `exclusions` cloud key. **Customer Basic Info RFC duplicate check** added — flags customers sharing a tax ID (same entity created twice) in an amber block below the upload box; exemptions for two RFCs and blank values, placeholders like `-` not exempt. **L&CAC new-customer matching switched from customer name to `Customer Code`** (exact, strict; no name fallback), with the SI upload now reporting lines missing a Customer Code |
| **2026-08 (Commission)** | New **left navigation** (Sales Report / Commission) wrapping the existing five tabs. **RAC commission** built: per-set Salesperson rule (`commRuleSP`, 35 items) + tiered BM rule (`commRuleBM`); BM = 👑 lead of any branch with a RAC-qty target, tier × branch RAC net; verified against real July SI data. Rule tables are cloud-synced, read-only with an Edit toggle. Per-person totals rounded to whole pesos |
| **2026-08 (Commission cont.)** | **Business group** (RAC-led / CAC-led) selector added per branch in Target Setup (`bizGroup`, per-month, English labels), used only by commercial commission. Target Setup now **opens read-only** with an Edit/Done toggle. **L&CAC (Commercial) commission** built (`commRuleCom`, two rate columns) with the **invoice-level model**: unit = one SI; tier locked to invoice month; brokerage allocated by `fee × (invoice line amount ÷ SO three-line total)`; incremental payout `full × collection% − paid-before`; four streams (RAC-SP / RAC-BM / L&CAC-SP / L&CAC-BM). **Sales Order Report** upload added (`Detail Sales Order` → per-SO RAC+LAC+CAC total, integer, upsert-merged into `soDenoms`). **Module 3 inputs** (`commBrokerage` / `commCollection` / `commPaid`) and **Post month to Commission-paid** added. China Projects join commission as virtual invoices using each row's `3rd Commission` / `Coll. Progress` / `SO Amt excl tax`. Negatives **floor to 0** (⚠ fee &gt; commission / ⚠ over-paid), never clawed back |
| **2026-08 (Commission polish)** | **Group-as-department**: when a branch has Target Setup groups, each group's lead is its BM and achievement/invoices are scoped to the group (RAC-BM and L&CAC-BM). Commission split into **three sub-tabs** (RAC / L&CAC / Inputs); detail columns renamed (SI Base, Com Full) and set to no-wrap; commission stream label COM → L&CAC. **Export Excel** added — one `.zip` of 1 + (n+1) three-sheet workbooks (Summary / Detail / SI Base) **plus a matching Summary PNG for each**, grouped departments partitioned by ◆ group; attribution by roster. Rule cards default collapsed |
| **2026-08-21 (Commission fix)** | **Commission-paid ledger sharded per month** (`commPaid_YYYY-MM`, one cell per month) to escape the Google Sheets 50,000-char single-cell limit that was silently dropping posted months (`no-cors` write + `doPost` 200-on-error hid the failure, so “saved ✓” was false). Load now **merges all month shards** and **auto-migrates** the legacy single `commPaid` cell (kept read-only as a fallback); Post / Unpost / Save write only the changed month shard and **read it back to verify** before reporting `Ledger saved & verified ✓` — a mismatch raises an explicit warning. Fossil duplicate settlement rows cleaned out of the ledger. **No Apps Script change** (kept generic to avoid a scope-change outage). Engine, Part 4, Post/Unpost and cross-month top-ups unchanged — only the load/save boundary shards on write and merges on read |
