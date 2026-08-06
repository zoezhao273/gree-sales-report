# 📊 GREE Mexico Sales Performance Report

A sales and collections performance reporting tool. Single-file HTML, no backend required, deployed on GitHub Pages. Roster, targets, and China Projects data are stored in Google Sheets. Five modules: **Collection Report**, **Sales Report**, **China Projects**, **Target Setup**, and **Upload Data**.

> Page title: `📊 Sales Performance Report` · `GREE Mexico`. The page opens on the **Upload Data** tab by default. All amounts are displayed in **MXN** (thousands separator, two decimal places).

---

## 🔗 Key Links

| Name | URL |
|------|-----|
| **Web app** | https://zoezhao273.github.io/greemexico-salesreport2/gree_sales_report2.html |
| **GitHub repository** | https://github.com/zoezhao273/greemexico-salesreport2 |
| **Apps Script URL** | https://script.google.com/macros/s/AKfycbyVug4IbhJ8IhVQySo2dZCHxlXlgCLhOAHeJYyZW6BA5HzKQhRam1uJ_L6t8AlDBxZU/exec |

> Uploading a new HTML file to the repository does not change any of these links.

---

## 📁 Files

| File | Description |
|------|-------------|
| `gree_sales_report2.html` (deployed name; local development file is `index.html`) | Complete single-file web app containing all modules |
| `README.md` | This documentation |

External libraries are loaded via CDN: SheetJS (Excel parsing) and html2canvas (PNG export).

---

## 🧭 Module Overview

| Tab | Purpose |
|-----|---------|
| **Collection Report** | Monthly and latest-day collections by department |
| **Sales Report** | Salesperson target achievement by month and branch, including Other Dept and All Branches Summary |
| **China Projects** | Manual entry of domestic China deals not in the ERP/SI data; automatically injected into Sales Report calculations |
| **Target Setup** | Manage monthly rosters, targets, and metric schemas |
| **Upload Data** | Upload three data sources: SI file, Customer Basic Info, Collection Doc |

---

## 1️⃣ Upload Data

Three upload cards side by side (stacked on narrow screens). Each card supports click or drag-and-drop. A **✕ Clear** button appears top-right when a file is loaded. On successful upload, the drop zone shows the filename and a summary. **All uploaded data is processed in browser memory only — nothing is sent to a server.**

| Card | Badge | File requirement | Key fields |
|------|-------|-----------------|------------|
| **Upload SI (Sales Invoice)** | for Sales Report | `.xlsx` with **one sheet per month** (e.g. `August 2026`) | See Data Processing Rules |
| **Upload Customer Basic Info** | for L&CAC Cus | `.xlsx` with columns `Customer Name` and `First Cooperate Date` | Used to calculate new L&CAC customer count |
| **Upload Collection Doc** | for Collection Report | `.xlsx` | `Doc Date`, `Department`, `Debit Amt in FC` |

- **SI file**: after upload the drop zone shows `{n} month(s) · {m} invoice lines · {k} orders (SO)`. SI represents invoiced, customer-signed deliveries — replacing the old Sales Order Report (order-entry basis, which included unshipped orders). A single SI file covers all months of the year; there is no need to upload files month by month.
- **Customer Basic Info**: shows `Customers {n}` after upload. Customers missing a First Cooperate Date are noted in muted text.
- **Collection Doc**: shows `{n} docs · {earliest date} → {latest date}`. Doc Date accepts `yyyy-mm-dd` text, native date cells, and Excel serial numbers.

> **SI parsing**: the app reads every sheet whose name resolves to a `yyyy-mm` month (format: English month name + year, e.g. `August 2026`) and normalises each row into a flat invoice line.

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

---

## 3️⃣ China Projects (Manual Entry)

Records domestic China delivery deals **not present in the ERP or SI data**. Entries are automatically converted to MXN and **injected into Sales Report calculations** (`computeP`), so single-branch views, Other Dept, All Branches Summary, YTD, MoM, CSV, and PNG all include this data automatically.

- **Attribution**: by **branch + employee ID** (`empBranch` / `empId`). The dropdown stores the Employee ID, not free text. Legacy rows without `empBranch` are attributed only when the ID is unambiguous (unique across the current month's roster); otherwise the row is skipped and a re-selection is prompted — preventing cross-branch double-counting.
- **Amount**: `Sales amount = SO Amt exclude tax × FE Rate` → MXN (rows in MXN use ×1). **Quantity = Qty. Product line = Line.**
- **Unbilled rows** (exclude-tax amount is blank or 0) contribute neither amount nor quantity.
- **Counting**: each project counts as **1 order and 1 customer** (deduplicated by project name). China projects are inherently valid orders and are not affected by any filters.
- **Default mode**: read-only (MXN). A toggle at the top switches to Edit mode (original currency).
- Data is stored under the `chinaProjects` key in Google Sheets. Click **☁️ Save to Cloud** to persist. The module has its own **🖼 Export PNG** (off-screen full-width table).

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

- Select a year and month. New months can be initialised by **📋 Copy from previous month** or **✨ Start blank** (copies only from the immediately preceding month — no chain search).
- The first row of each department is a read-only auto-sum total. 👑 marks team leads (optional targets). Row order and column widths are drag-adjustable.
- **Employee IDs must be unique and must match the `Salesman Code` (employee number) in the SI file** ⭐:
  - Attribution now relies entirely on the employee ID. If a roster ID does not match the SI Salesman Code, that person's orders will fall into Other Dept.
  - Duplicate IDs within the same branch cause data to be merged incorrectly; a validation check runs before saving.
- **Metric schema** controls which columns appear in the Sales Report. Each metric is `{key, label, unit, src}`:
  - `unit`: `qty` (quantity) or `amt` (amount excl. tax)
  - `src`: `RAC` / `LAC` / `CAC` / `LAC+CAC` (combined L&CAC amount) / `CUST` (new L&CAC customers — requires Customer Basic Info upload)
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

### YTD calculation
- Accumulated from January of the current year through the selected month.
- Each month uses **its own roster** for attribution. If a month has no roster, the **previous month's roster** is used. If neither exists, that month is excluded from YTD.

### Month identification
- Sales: **sheet name** takes precedence (`August 2026` → `2026-08`). `Invoice Date` (`yyyy-mm-dd`) is used for display/validation only.
- Collections: `Doc Date` (`yyyy-mm-dd`, plus native date cells and Excel serial numbers).

---

## ☁️ Cloud Sync

### Storage architecture

| Data | Location | Key |
|------|----------|-----|
| Monthly rosters | Google Sheets | `rosters` |
| Monthly targets | Google Sheets | `targets` |
| Monthly metric schemas | Google Sheets | `schemas` |
| China Projects | Google Sheets | `chinaProjects` |
| Column-width preferences | Browser localStorage | Local only, not synced |
| SI / Customer / Collection Excel files | Not stored | Processed in memory on each upload |

> Legacy `roster` key compatibility: if the sheet has the old single-key `roster` but no `rosters`, it is automatically mapped to the current month on load.

### Google Sheets structure
Data is stored in a sheet named `targets`. Each row is one key-value pair; the value is a JSON string:

| Column A (key) | Column B (value) |
|----------------|-----------------|
| `rosters` | `{"2026-07":[{branch, people:[{id, name, isManager, group}]}], ...}` |
| `targets` | `{"2026-07":{"Mexico City Branch":{"MX250024":{"RAC":100,"LAC":200000,"CAC":2}}}, ...}` |
| `schemas` | `{"2026-07":[{"key":"RAC","label":"RAC","unit":"qty","src":"RAC"}, ...], ...}` |
| `chinaProjects` | `[{month, feRate, empId, empBranch, project, currency, line, qty, soIncl, collProg, soExcl, thirdComm, note, progress}, ...]` |

### Read / write flow
- **Read (page load):** GET request to the Apps Script URL; parses `rosters`, `targets`, `schemas`, and `chinaProjects`.
- **Write (Save to Cloud):** POST each key in sequence using `no-cors` mode to avoid CORS preflight issues and URL length limits.

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
After any code change: open the project at script.google.com → Save (Cmd+S) → Deploy → Manage deployments → pencil icon → set version to "New version" → Deploy. **The deployment URL does not change; `GS_URL` in the HTML does not need to be updated.**

### Troubleshooting
| Symptom | Likely cause / fix |
|---------|-------------------|
| Save failed / Failed to fetch | Apps Script not re-deployed after a code change, or network/auth issue. Open `GS_URL` directly in a browser and check for a JSON response. |
| Cloud unavailable — offline mode | The app still works; data is in local memory only. Reload after restoring network access. |
| Changes not taking effect | Browser or CDN has cached the old HTML. Hard-refresh with `Cmd+Shift+R`. |

---

## 📤 Export Reference

| Module | PNG | CSV |
|--------|-----|-----|
| Sales Report — single branch / Other Dept | ✅ | ✅ |
| All Branches Summary | ✅ (full width) | ✅ (ALL BRANCHES row includes Other Dept) |
| China Projects | ✅ (off-screen full-width table) | — |
| Collection Report | ✅ | ✅ |

- **Export All PNG**: batch-exports all rostered branches + Other Dept + All Branches Summary for the selected month. A floating progress overlay survives report DOM rebuilds during the batch loop.
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

---

## 🛠️ Technical Notes

- Pure front-end single-file HTML. All logic runs in the browser; works via `file://` and GitHub Pages alike.
- External dependencies: SheetJS (Excel parsing) and html2canvas (PNG export), both loaded from CDN.
- Cloud layer: Google Apps Script (POST write / GET read) + Google Sheets.
- Hosting: GitHub Pages (public repository, free permanent URL).
- Amount formatting: `toLocaleString('es-MX')`, two decimal places (MXN).

---

## 📝 Changelog

| Date | Changes |
|------|---------|
| 2026-05-13 | Initial release; Google Sheets cloud storage integrated |
| 2026-05-31 | Roster/targets changed to per-month storage; Branch Summary added (MoM, YTD); cloud writes switched to POST; Customers column added; PNG export truncation fixed |
| 2026-07 | Branch Summary merged into Report branch dropdown as All Branches Summary; three exclude toggles defaulted on, Lock Stock and Inner Use merged; valid order rule added (must contain RAC/LAC/CAC line); Other Dept dimension added (dropdown entry + Summary last row); Collection Report module added (MTD + Latest-Date, Finance→Pending, dept sort); upload area redesigned to three-card compact layout with Clear buttons and in-card status; Report renamed to Sales Report; all amounts unified as MXN |
| 2026-07 (subsequent) | **China Projects** manual-entry module added (branch+ID attribution, SO Amt × FE Rate → MXN, unbilled rows skipped, 1 order + 1 customer per project, injected into computeP); fixed manager placeholder ID cross-branch data injection (composite branch+ID key); Collection Doc parser upgraded to auto-detect header row + column name aliases; Export All PNG batch export added |
| **2026-08** | **Sales data source switched from Sales Order Report (order-entry) to SI — Sales Invoice (delivered and signed-off)**. Single multi-sheet file; month = sheet name. **Product line now determined by Item Code prefix** (`MXR→RAC` / `MXL→LAC` / `MXC→CAC`; all others including MXI not counted). **Order unit changed to SO Number** (multi-invoice SO deduplication). **Salesperson attribution switched to employee ID** (Salesman Code exact match; name fuzzy fallback). **Three ERP exclude filters removed**. Cross-month SOs use **Plan B** (amounts by invoice month; order/customer counted once in earliest month). Amount column: `Sub Amount After Discount Without Tax`. **All Branches Summary grand total now includes Other Dept** (table and CSV), making the total row equal the sum of all rows beneath it |
