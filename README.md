# 📊 GREE Mexico Sales Performance Report

销售业绩报表工具，支持上传 Sales Order Report、按月份名册自动匹配业务员、计算目标完成率，并提供分公司汇总视图和年度累计数据。

---

## 🔗 重要链接

| 名称 | 地址 |
|------|------|
| **网页地址** | https://greemexico-salesreport2.netlify.app/ |
| **GitHub 仓库** | https://github.com/zoezhao273/greemexico-salesreport2 |
| **Apps Script URL** | https://script.google.com/macros/s/AKfycbyVug4IbhJ8IhVQySo2dZCHxlXlgCLhOAHeJYyZW6BA5HzKQhRam1uJ_L6t8AlDBxZU/exec |

---

## 📁 文件说明

| 文件 | 说明 |
|------|------|
| `gree_sales_report2.html` | 完整网页，包含所有功能，单文件无依赖 |
| `README.md` | 项目说明文档 |

---

## ✨ 功能模块

### 1. Upload Data
上传从 ERP 导出的 Sales Order Report（`.xlsx`），支持点击或拖拽上传。文件需包含两个 Sheet：
- `Sales Order` — 订单主表
- `Detail Sales Order` — 订单明细（含产品线和金额）

上传后数据仅保留在当前浏览器内存中，不会上传到任何服务器。

---

### 2. Target Setup（月度名册 + 目标设置）

**核心概念：名册和目标按月份独立存储。**

每个月可以有不同的人员组成，适用于人员入职、离职、调岗等情况：
- 某人在 4 月有目标 → 4 月报表显示该人；5 月没有设置 → 5 月不显示
- 增删人员只影响当前选中月份，不影响其他月份

**操作步骤：**
1. 选择年份和月份
2. 若该月尚无名册，系统会提示：
   - **📋 Copy from 上月** — 复制上月名册作为起点（推荐）
   - **✨ Start blank** — 从空白开始
   - 若上月也没有名册，则只显示 Start blank，并提示先维护上月
3. 在编辑器中增删人员、修改 Employee ID 和姓名
4. 填写各人的月度目标（RAC 填数量，LAC/CAC 填不含税金额）
5. 点击 **☁️ Save to Cloud** 保存

**注意事项：**
- 每个部门第一行为汇总行（自动求和，不可编辑）
- 👑 标记为分总，目标可选填
- Employee ID **必须唯一**，不同人员使用相同 ID 会导致数据合并计算（数据错误）
- 可拖拽行来调整人员顺序，可拖拽列边缘调整列宽

---

### 3. Report（业务员明细报表）

按月份和分公司查看每位业务员的目标完成情况。

**筛选项：**
| 筛选 | 默认 | 说明 |
|------|------|------|
| Exclude Deleted | ✅ 开启 | 排除 ERP Order Status = Deleted 的订单 |
| Exclude Lock Stock | ✅ 开启 | 排除 Order Type = Lock Stock Order 的订单 |
| Exclude Not-Exist | ❌ 关闭 | 排除 ERP Order Status = Not-Exist 的订单 |

**报表结构：**
- 第一行：部门汇总（Target 合计 / Progress 合计 / 整体完成率）
- 以下各行：分总 + 各业务员
- 完成率颜色：🟢 ≥100%　🟡 50–99%　🔴 <50%

**导出：** 支持导出 PNG 图片和 CSV 文件。

---

### 4. Branch Summary（分公司汇总报表）

横向对比所有分公司的当月业绩，包含：

| 列组 | 内容 |
|------|------|
| Customers | 当月有订单的唯一客户数 |
| Orders | 当月订单数 |
| RAC | Target(qty) · Qty · Amt · Achv% · Qty(上月) · MoM% |
| LAC | Target(amt) · Qty · Amt · Achv% · Amt(上月) · MoM% |
| CAC | Target(amt) · Qty · Amt · Achv% · Amt(上月) · MoM% |
| Total | 当月三线合计 Qty + 合计 Amt |
| YTD | 年初至今各线 Qty/Amt + 合计（仅统计有名册的月份） |

MoM% 颜色：▲绿色（环比增长）▼红色（环比下降）

**导出：** 支持导出完整宽度的 PNG 图片（含所有列）和 CSV 文件。

---

## 🏗️ 数据处理规则

### 业务员归属
订单归属以**名册**为准，与 ERP 中的 Branch 字段无关。例如：Norka 在 Merida Branch 产生的订单，因其在名册中属于 Mexico City Branch，自动归入 Mexico City Branch 统计。

匹配逻辑（优先级从高到低）：
1. 名字完全一致
2. 名字互相包含（子字符串匹配）
3. 关键词重叠（超过 2 个字符的词，取重叠最多的）

### 产品线识别
取 Detail Sales Order 中 `Category Code` 的第 3-5 位：
- `RAC` → 统计销售数量（Qty）和不含税销售额（Amt）
- `LAC` → 统计不含税销售额（Amt）和数量
- `CAC` → 统计不含税销售额（Amt）和数量
- 其他（SRV 等）→ 不纳入统计

### YTD 计算规则
- 从当年 1 月累计到所选月份
- 每个月使用**该月自己的名册**计算归属
- 若某月没有名册，则使用**上月名册**代替
- 若某月及上月都没有名册，该月数据**不计入** YTD（不用更早的名册填充）

---

## ☁️ 云端同步详解

### 存储架构

| 数据类型 | 存储位置 | 说明 |
|----------|----------|------|
| 各月名册（rosters） | Google Sheets | 按月份独立存储，key = `rosters` |
| 各月目标（targets） | Google Sheets | 按月份独立存储，key = `targets` |
| 列宽偏好 | 浏览器 localStorage | 本地生效，各人独立，不同步 |
| 销售数据（Excel） | 不存储 | 每次本地上传，仅在内存中处理 |

### Google Sheets 数据结构

数据存储在 Google Sheets 的 `targets` Sheet 中，每行为一个 key-value 对：

| A 列（key） | B 列（value） |
|-------------|---------------|
| `rosters` | `{"2026-04": [...branches], "2026-05": [...branches]}` |
| `targets` | `{"2026-04": {"Mexico City Branch": {"MX250024": {"RAC": 50, ...}}}, ...}` |

### 读写流程

**读取（页面加载时自动执行）：**
- 发送 GET 请求到 Apps Script URL
- Apps Script 读取 Google Sheets 返回完整数据
- 页面解析并渲染名册和目标

**写入（点击 Save to Cloud）：**
- 收集当前编辑器中的名册和目标数据
- 分两次 POST 请求写入：先存 `rosters`，再存 `targets`
- 使用 `no-cors` 模式发送，避免浏览器 CORS preflight 限制
- 写入成功后顶部状态栏显示 "Saved ✓ HH:MM:SS"

### Apps Script 代码

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
    const payload = JSON.parse(e.postData.contents);
    const { key, value } = payload;
    setData(key, value);
    result = { ok: true };
  } catch(err) {
    result = { ok: false, error: err.message };
  }
  return ContentService
    .createTextOutput(JSON.stringify(result))
    .setMimeType(ContentService.MimeType.JSON);
}

function getAllData() {
  const sheet = getOrCreateSheet();
  const data = sheet.getDataRange().getValues();
  const result = {};
  data.forEach(row => {
    if (row[0]) {
      try { result[row[0]] = JSON.parse(row[1]); }
      catch(e) { result[row[0]] = row[1]; }
    }
  });
  return result;
}

function setData(key, value) {
  const sheet = getOrCreateSheet();
  const data = sheet.getDataRange().getValues();
  for (let i = 0; i < data.length; i++) {
    if (data[i][0] === key) {
      sheet.getRange(i + 1, 2).setValue(JSON.stringify(value));
      return;
    }
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

### 重新部署 Apps Script 的步骤

每次修改 Apps Script 代码后必须重新部署才能生效：

1. 打开 [script.google.com](https://script.google.com)，进入项目
2. 修改代码后点 **保存（Cmd+S）**
3. 点右上角 **部署 → 管理部署**
4. 点铅笔图标（编辑现有部署）
5. 版本选择 **"新建版本"**
6. 点 **部署**

> ⚠️ 部署 URL 不会改变，无需修改 HTML 中的 `GS_URL`。

### 常见问题排查

**Save failed: Failed to fetch**

| 可能原因 | 排查方法 | 解决方案 |
|----------|----------|----------|
| Apps Script 未重新部署 | 直接访问 GS_URL，看是否返回 JSON | 按上述步骤重新部署 |
| 网络问题 | 换网络或关闭 VPN 重试 | 切换网络 |
| Google 账号需要授权 | 直接访问 GS_URL，看是否跳转登录页 | 重新授权后部署 |

**云端状态显示"Cloud unavailable — offline mode"**

页面仍可正常使用，数据只在本地内存中。恢复网络后重新打开页面即可同步。

---

## 🔄 更新流程

### 更新网页功能
1. 在 Claude 对话中描述需要修改的内容
2. 下载新的 `gree_sales_report2.html`
3. 打开 GitHub 仓库，上传覆盖旧文件（Add file → Upload files）
4. 底部点 **Commit changes**
5. 约 1 分钟后 GitHub Pages 自动更新，**链接不变**

### 日常维护目标数据
1. 打开网页 → **Target Setup**
2. 选择年份和月份
3. 若是新月份，点 **📋 Copy from 上月** 后按实际情况增删人员
4. 填写各人目标数字
5. 点击 **☁️ Save to Cloud**
6. 其他人刷新页面即可看到最新数据

---

## 🛠️ 技术说明

- 纯前端单文件 HTML，无需服务器，所有逻辑在浏览器中运行
- 依赖库（均通过 CDN 加载，无需安装）：
  - SheetJS — Excel 文件解析
  - html2canvas — PNG 导出
- 云端存储：Google Apps Script（POST 写入 / GET 读取）+ Google Sheets
- 部署：GitHub Pages（Public 仓库，免费永久链接）
- 本地打开（`file://`）与 GitHub Pages 均可正常使用

---

## 📝 维护记录

| 日期 | 更新内容 |
|------|----------|
| 2026-05-13 | 初始版本上线，接入 Google Sheets 云端存储 |
| 2026-05-31 | 名册和目标改为按月独立存储；新增 Branch Summary 视图（含 MoM、YTD）；云端写入改为 POST 解决 URL 超长问题；新增 Customers 列；修复 PNG 导出截断问题 |
