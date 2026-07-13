# 📊 GREE Mexico Sales Performance Report

销售与收款业绩报表工具。单文件 HTML，无后端依赖，部署在 GitHub Pages，名册/目标数据存于 Google Sheets 云端。支持四个模块：**Collection Report（收款报表）**、**Sales Report（销售报表）**、**Target Setup（名册与目标）**、**Upload Data（数据上传）**。

> 界面顶部标题：`📊 Sales Performance Report` · `GREE Mexico`。页面加载后默认停在 **Upload Data** 页。金额统一以 **MXN** 显示（千分位、两位小数）。

---

## 🔗 重要链接

| 名称 | 地址 |
|------|------|
| **网页地址** | https://zoezhao273.github.io/greemexico-salesreport2/gree_sales_report2.html |
| **GitHub 仓库** | https://github.com/zoezhao273/greemexico-salesreport2 |

> 若把新版 HTML 上传覆盖到仓库，链接不变。

---

## 📁 文件说明

| 文件 | 说明 |
|------|------|
| `gree_sales_report2.html`（部署文件名，本地开发文件为 `index.html`） | 完整网页，包含全部四个模块，单文件无依赖 |
| `README.md` | 本说明文档 |

依赖库均通过 CDN 加载：SheetJS（Excel 解析）、html2canvas（PNG 导出）。

---

## 🧭 模块总览

页面顶部四个 Tab，顺序如下：

| Tab | 作用 |
|-----|------|
| **Collection Report** | 按月展示各部门的月度累计收款与最新一天收款 |
| **Sales Report** | 按月/分公司展示业务员目标达成，含 Other Dept 与 All Branches Summary |
| **Target Setup** | 维护每月名册、目标、指标口径（schema） |
| **Upload Data** | 上传三类数据源（销售、客户基础信息、收款） |

---

## 1️⃣ Upload Data（数据上传）

三张上传卡并排（窄屏自动堆叠）。每张卡都可点击或拖拽上传，右上角有 **✕ Clear** 清除按钮（未上传时禁用），上传成功后虚线框内底部显示两行：文件名 + 读取结果。**所有上传数据只保留在当前浏览器内存中，不上传服务器。**

| 卡片 | 标签 | 文件要求 | 关注字段 |
|------|------|----------|----------|
| **Upload Sales Order Report** | for Sales Report | `.xlsx`，含 `Sales Order` 和 `Detail Sales Order` 两个 Sheet | 见下方数据处理规则 |
| **Upload Customer Basic Info** | for L&CAC Cus | `.xlsx`，含列 `Customer Name`、`First Cooperate Date` | 用于计算 L&CAC 新客户开发数 |
| **Upload Collection Doc** | for Collection Report | `.xlsx` | `Doc Date`、`Department`、`Debit Amt in FC` |

- 销售文件上传后，虚线框显示 `Sales Orders {n} · Detail Rows {m}`。
- 客户文件上传后显示 `Customers {n}`（若有客户缺 First Cooperate Date 会淡色标注）。
- 收款文件上传后显示 `{n} docs · {最早日期} → {最晚日期}`。收款 Doc Date 兼容 `yyyy-mm-dd` 文本、真正的日期单元格与 Excel 序列号。

---

## 2️⃣ Sales Report（销售报表）

按 **月份** 和 **分公司** 查看每位业务员的目标完成情况。

### 分公司下拉框（Branch）
从上到下依次为：
1. 各名册分公司（如 Mexico City Branch、Monterrey Branch、Project Dept）
2. **Other Dept**（业务员不在任何名册中的订单，详见数据处理规则）
3. **All Branches Summary**（横向对比全部分公司的汇总视图）

### 过滤开关（三个，默认全部开启 ✅）
| 开关 | 默认 | 判定条件 |
|------|------|----------|
| Exclude Deleted | ✅ | `ERP Order Status = Deleted` |
| Exclude Lock Stock / Inner Use | ✅ | `Order Type = Lock Stock Order` **或** `Inner Use` |
| Exclude Not-Exist | ✅ | `ERP Order Status = Not-Exist`（对应 Draft 草稿单） |

> 三个开关共享于 Sales Report 的所有分公司视图（含 Other Dept 与 All Branches Summary）。

### 单个分公司视图
- 第一行：分公司汇总（`∑ {branch} Total`）
- 之后：分总（👑）/分组小计 + 各业务员
- 每个指标（由 schema 定义）显示 **Target / Progress / Achv%**
- 达成率颜色：🟢 ≥100%　🟡 50–99%　🔴 <50%

### Other Dept 视图
业务员经名册三级匹配后匹配不到任何名册人员的订单，统一归入此处。
- 结构为扁平表：`∑ Other Dept Total` 汇总行 + 每个业务员一行，业务员名字前带一列 **Dept**（原始 Branch 字段），同部门相邻，无部门小计。
- 因无目标，**删除 Target 和 Achv% 列**，仅显示各指标 Progress。

### All Branches Summary 视图
横向对比所有分公司，每行列组包含：
| 列组 | 内容 |
|------|------|
| Customers / Orders | 有效订单的唯一客户数、订单数 |
| RAC | Target(qty) · Qty · Amt · Achv% · Qty(上月) · MoM% |
| LAC | Target(amt) · Qty · Amt · Achv% · Amt(上月) · MoM% |
| CAC | Target(amt) · Qty · Amt · Achv% · Amt(上月) · MoM% |
| Total | 当月三线合计 Qty + 合计 Amt |
| YTD | 年初至今各线 Qty/Amt + 合计（仅统计有名册的月份） |

- 首行 `🌐 All Branches` 为总计，**仅统计名册分公司**。
- 末行 **Other Dept** 作为独立维度单列，样式与分公司行一致；因无目标，Target/Achv% 显示 `—`，其余 Qty/Amt/MoM/YTD 正常计算。
- MoM% 颜色：▲绿色（环比增长）▼红色（环比下降）。

**导出：** Sales Report 各视图均支持 PNG 与 CSV 导出（All Branches Summary 的 PNG 为完整宽度）。

---

## 3️⃣ Collection Report（收款报表）

按月展示各部门收款情况，用于每日跟踪当月累计到账与最新一天到账。

- 顶部选择 **月份**（格式 `yyyy-mm`），来源为收款文件中出现过的月份。
- 表格两个维度并列成两列：
  - **Month-to-Date**：所选月份的累计收款
  - **Latest-Date**：所选月份内**最后一个 Doc Date** 当天的收款（看当月时即等于文件最后一天=今天；看过去月份则为该月最后一天，不串月）
- 每列都有 `∑ Total` 合计行 + 各部门行。
- 金额以 **MXN** 显示。
- **导出：** 报表右上角提供 PNG / CSV。

### 部门处理规则
- **Finance Dept** 的部门名显示为 **Pending**。
- 部门排序：**Target Setup 中的部门（按名册先后顺序）→ 其他部门（字母序）→ Pending 放在最末**。

---

## 4️⃣ Target Setup（每月名册 + 目标 + 指标口径）

**核心概念：名册、目标、指标口径（schema）都按月份独立存储。**

- 选择年份和月份；新月份可 **📋 Copy from 上月** 或 **✨ Start blank**（仅从上一月复制，不向更早月份链式查找）。
- 每个部门第一行为汇总行（自动求和，不可编辑）；👑 为分总，目标可选填；可拖拽调整人员顺序与列宽。
- **Employee ID 必须唯一**：同一分公司内不同人员使用相同 ID 会导致数据合并计算（错误），保存前会校验。
- **指标口径（schema）** 决定 Sales Report 展示哪些列。每个指标为 `{key,label,unit,src}`：
  - `unit`：`qty`（数量）或 `amt`（不含税金额）
  - `src`：`RAC` / `LAC` / `CAC` / `LAC+CAC`（合并 L&CAC 金额）/ `CUST`（L&CAC 新客户数，需上传 Customer Basic Info 才能计算）
  - 默认口径：RAC(qty)、LAC(amt)、CAC(amt)
  - 示例（2026-07）：RAC 数量、L&CAC 合并金额、L&CAC 新客户数
- 点击 **☁️ Save to Cloud** 保存名册 / 目标 / 指标口径。

---

## 🏗️ 数据处理规则（Sales Report）

### 业务员归属
订单归属以**名册**为准，与 ERP 中的 Branch 字段无关。匹配逻辑（优先级从高到低）：
1. 名字完全一致
2. 名字互相包含（子字符串匹配）
3. 关键词重叠（长度 >2 的词，取重叠最多者，需重叠 ≥2）

> 名册里 1–6 月常用全名、销售表用简称的历史月份，靠第 2/3 级模糊匹配兜住；建议名册尽量与销售表 Salesman 字段保持一致的写法，最稳。

### 产品线识别
取 Detail Sales Order 中 `Category Code` 的第 3-5 位（如 `1-RAC-01` → `RAC`）：
- `RAC` → 统计数量（Qty）和不含税金额（Amt）
- `LAC` / `CAC` → 统计不含税金额（Amt）和数量
- 其他（`IST` 安装 / `PAT` 配件 / `SRV`/`SMP` 服务等）→ **不纳入统计**

### 有效订单（计数口径）⭐
一个订单**只有包含至少一条 RAC/LAC/CAC 明细时**，才计入 **Orders 数和 Custs 数**。纯配件 / 服务 / 安装单（只有 PAT/SRV/IST 等）不计入订单数与客户数（它们对三线金额本就贡献为 0）。此规则同时作用于单分公司、Other Dept、All Branches Summary 与 Collection 无关。

### 金额与数量
- 金额取 Detail 的 `Final Sub Total Without Tax`（不含税）
- 数量取 Detail 的 `Quantity`

### Other Dept 归属
业务员经三级匹配后匹配不到任何名册人员（视为不在 Target Setup 中），其订单归入 Other Dept，按原始 `Branch` 字段分部门、部门内按业务员，仅统计有效订单，无目标。

### YTD 计算规则
- 从当年 1 月累计到所选月份
- 每个月使用**该月自己的名册**计算归属；若该月无名册则用**上月名册**代替；若该月及上月都无名册，则该月不计入 YTD

### 月份识别
- 销售 `Order Date`：`DD-MM-YYYY`（如 `08-07-2026` → `2026-07`）
- 收款 `Doc Date`：`yyyy-mm-dd`（同时兼容日期单元格与 Excel 序列号）

---

## ☁️ 云端同步

### 存储架构

| 数据类型 | 存储位置 | Key |
|----------|----------|-----|
| 各月名册 | Google Sheets | `rosters` |
| 各月目标 | Google Sheets | `targets` |
| 各月指标口径 | Google Sheets | `schemas` |
| 列宽偏好 | 浏览器 localStorage | 本地生效，不同步 |
| 销售 / 客户 / 收款 Excel | 不存储 | 每次本地上传，仅内存处理 |

> 兼容旧的单一 `roster` key：若云端只有旧 `roster` 而无 `rosters`，加载时会自动映射到当前月。

### Google Sheets 数据结构
数据存于 `targets` Sheet，每行一个 key-value 对，value 为 JSON 字符串：

| A 列（key） | B 列（value） |
|-------------|---------------|
| `rosters` | `{"2026-07":[{branch,people:[{id,name,isManager,group}]}], ...}` |
| `targets` | `{"2026-07":{"Mexico City Branch":{"MX250024":{"RAC":100,"LAC":200000,"CAC":2}}}, ...}` |
| `schemas` | `{"2026-07":[{"key":"RAC","label":"RAC","unit":"qty","src":"RAC"}, ...], ...}` |

### 读写流程
- **读取（页面加载）：** GET 请求到 Apps Script URL，解析 `rosters`/`targets`/`schemas` 并渲染。
- **写入（Save to Cloud）：** 依次 POST 写入 `rosters`、`targets`、`schemas`，使用 `no-cors` 模式避免 CORS 预检和 URL 超长问题。

### Apps Script 代码（通用 key-value，自动兼容 schemas）
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
  data.forEach(row => { if (row[0]) { try { result[row[0]] = JSON.parse(row[1]); } catch(e) { result[row[0]] = row[1]; } } });
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

### 重新部署 Apps Script
每次改动代码后：script.google.com 打开项目 → 保存（Cmd+S）→ 部署 → 管理部署 → 铅笔图标 → 版本选"新建版本" → 部署。**部署 URL 不变，无需改 HTML 中的 `GS_URL`。**

### 常见排查
- **Save failed / Failed to fetch**：多为 Apps Script 未重新部署，或网络/授权问题。直接访问 GS_URL 看是否返回 JSON。
- **Cloud unavailable — offline mode**：页面仍可用，数据只在本地内存；恢复网络后重新打开页面即可同步。

---

## 📤 导出说明

| 模块 | PNG | CSV |
|------|-----|-----|
| Sales Report（单分公司 / Other Dept） | ✅ | ✅ |
| All Branches Summary | ✅（完整宽度） | ✅（含 Other Dept 行） |
| Collection Report | ✅ | ✅ |

PNG 导出宽表时会临时移除 `overflow-x` 限制并展开容器宽度，再调用 html2canvas，最后在 `.then()` 与 `.catch()` 中恢复样式。

---

## 🔄 更新流程

### 更新网页功能
1. 在对话中描述修改需求，下载新的 HTML
2. GitHub 仓库 → Add file → Upload files → 覆盖旧文件 → Commit changes
3. 约 1 分钟后 GitHub Pages 自动更新，**链接不变**

### 日常维护
- **目标数据：** Target Setup → 选月份 →（新月份先 Copy from 上月）→ 填目标 → Save to Cloud
- **出报表：** Upload Data 上传当日导出的 Sales Order Report / Collection Doc（如需 L&CAC 新客户再传 Customer Basic Info）→ 到对应报表 Tab 选月份查看/导出

---

## 🛠️ 技术说明

- 纯前端单文件 HTML，所有逻辑在浏览器运行，`file://` 与 GitHub Pages 均可用
- 依赖：SheetJS（Excel 解析）、html2canvas（PNG 导出），均 CDN 加载
- 云端：Google Apps Script（POST 写 / GET 读）+ Google Sheets
- 部署：GitHub Pages（Public 仓库，免费永久链接）
- 金额格式：`toLocaleString('es-MX')`，两位小数（MXN）

---

## 📝 维护记录

| 日期 | 更新内容 |
|------|----------|
| 2026-05-13 | 初始版本上线，接入 Google Sheets 云端存储 |
| 2026-05-31 | 名册/目标改为按月独立存储；新增 Branch Summary（含 MoM、YTD）；云端写入改 POST；新增 Customers 列；修复 PNG 导出截断 |
| 2026-07 | Branch Summary 合并进 Report 的分公司下拉（All Branches Summary）；三个排除开关默认全开、Lock Stock 与 Inner Use 合并；新增"有效订单"计数口径（须含 RAC/LAC/CAC 明细）；新增 Other Dept 维度（分公司下拉项 + Summary 末行）；新增 Collection Report 收款报表模块（MTD + Latest-Date、Finance→Pending、部门排序）；上传区改为三卡紧凑布局 + Clear 按钮 + 框内状态；Report 更名为 Sales Report；金额口径统一显示为 MXN |
