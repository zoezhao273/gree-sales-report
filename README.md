# 📊 GREE Mexico Sales Performance Report

销售业绩报表工具，支持上传 Sales Order Report、自动匹配业务员、计算任务完成率。

---

## 🔗 重要链接

| 名称 | 地址 |
|------|------|
| **网页地址** | https://zoezhao273.github.io/greemexico-salesreport2/gree_sales_report.html |
| **GitHub 仓库** | https://github.com/zoezhao273/greemexico-salesreport2 |
| **Google Sheets（目标数据）** | *(在此填入你的 Google Sheet 链接)* |
| **Apps Script URL** | https://script.google.com/macros/s/AKfycbyVug4IbhJ8IhVQySo2dZCHxlXlgCLhOAHeJYyZW6BA5HzKQhRam1uJ_L6t8AlDBxZU/exec |

---

## 📁 文件说明

| 文件 | 说明 |
|------|------|
| `gree_sales_report.html` | 完整网页，包含所有功能 |
| `README.md` | 项目说明文档 |

---

## ✨ 功能介绍

- **Upload Data** — 上传 Sales Order Report（.xlsx），支持拖拽
- **Target Setup** — 设置各分公司、各业务员的月度目标（RAC/LAC/CAC），保存到云端
- **Report** — 按月份、分公司查看销售完成情况，支持排除 Deleted 订单和 Lock Stock 订单

### 报表结构
- 第一行：部门汇总（Total）
- 第二行起：分总（无强制考核，可选填目标）+ 各业务员
- 完成率颜色：🟢 ≥100%　🟡 50–99%　🔴 <50%

### 数据处理规则
- 按名册归属识别业务员，与 ERP 中的分公司字段无关（例如 Norka 在 Merida Branch 产生的订单，自动归入 Mexico City Branch）
- 产品线取 Category Code 第3-5位：RAC 统计销售数量，LAC/CAC 统计不含税销售额
- SRV 等其他品类不纳入统计范围

---

## 🗂️ 组织架构（名册）

### Mexico City Branch
| ID | 姓名 | 角色 |
|----|------|------|
| MGR | Gesi Tian | 分总 |
| MX250024 | Gabriela Chavez Perez | 业务员 |
| MX250027 | Norka Yamile Aranda UC | 业务员 |
| MX260007 | Adrian Cabrera Ramirez | 业务员 |
| MX260008 | Ahumada Armenta Hugo Armando | 业务员 |
| MX260009 | Artiguez Utrera Maria del Rosario | 业务员 |

### Monterrey Branch
| ID | 姓名 | 角色 |
|----|------|------|
| MGR | Branch Manager | 分总 |
| MX250022 | Olam Yhoshua Arriaga Segovia | 业务员 |
| MX260003 | Maria de Los Angeles de Haro Grimaldo | 业务员 |
| MX260015 | Daniel de La Torre Gonzalez | 业务员 |

### Project Dept
| ID | 姓名 | 角色 |
|----|------|------|
| MGR | Project Manager | 分总 |
| MX250006 | Sebastián | 业务员 |
| MX250025 | Miguel Angel Heredia Marquez | 业务员 |
| MX250026 | Noemi Marina Alvarez Moreno | 业务员 |

---

## ☁️ 数据存储说明

| 数据类型 | 存储位置 | 说明 |
|----------|----------|------|
| 名册 + 月度目标 | Google Sheets | 云端共享，所有人打开网页自动读取 |
| 列宽偏好 | 浏览器 localStorage | 本地生效，各人独立 |
| 销售数据（Excel） | 不上传 | 每次本地上传，仅在当前浏览器内存中处理 |

---

## 🔄 更新流程

### 更新网页功能
1. 在 Claude 对话中描述需要修改的内容
2. 下载新的 `gree_sales_report.html`
3. 在 GitHub 仓库中上传覆盖旧文件
4. GitHub Pages 约 1 分钟后自动更新，**链接不变**

### 更新目标数据
1. 打开网页 → **Target Setup**
2. 选择月份，填写各人目标数字
3. 点击 **☁️ Save to Cloud**
4. 其他人刷新页面即可看到最新数据

---

## 🛠️ 技术说明

- 纯前端单文件 HTML，无需服务器，可本地直接打开使用
- 依赖库：SheetJS（Excel 解析）、html2canvas（PNG 导出），均通过 CDN 加载
- 云端存储：Google Apps Script + Google Sheets（GET 请求，无 CORS 问题）
- 部署：GitHub Pages（Public 仓库，免费永久链接）

---

## 📝 维护记录

| 日期 | 更新内容 |
|------|----------|
| 2026-05-13 | 初始版本上线，接入 Google Sheets 云端存储 |
