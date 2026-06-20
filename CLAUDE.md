# CLAUDE.md

> AI Agent 指令文件。执行任务前请优先查阅 `.claude/skills/` 下各 Skill 定义，获取完整的交互流程与业务规则。

## 项目概述

东风奕派 EPS（电子采购系统）价格合规检查项目。基于 pandas 对 `eps3_spare.xlsx`（备件输机）与 `eps3_mass.xlsx`（量产件输机）进行交叉比对，覆盖联动标识、价格差异、物流方式、商折识别、工程设变跟踪、工装分摊剥离六类检查。

分析结果可导出 Excel、追加至飞书在线表格、发送飞书群消息提醒，或借助 PowerPoint MCP 生成汇报幻灯片。Power BI MCP 可辅助数据建模与可视化。

核心数据源：
- `data/eps3_spare.xlsx` — 备件输机价格（约 15.7 万行，40 列）
- `data/eps3_mass.xlsx` — 量产件输机价格（约 17.3 万行，30 列）

## 检查能力总览（8 个 Notebook，6 大类）

### 一、日常合规点检（4 个）

| # | 检查项 | Notebook | 输入 | 输出 | 飞书 |
|---|--------|----------|------|------|------|
| 1 | 94 项联动标识检查 | `94项需改为联动标识检查.ipynb` | spare + 不联动清单 | `联动标识检查结果/` | — |
| 2 | 160 项量产件跟踪 | `160项量产件检查跟踪_mass.ipynb` | mass + 不联动清单 | `联动标识检查结果/` | — |
| 3 | 备件输机未联动 | `备件输机(应联动)_未联动检查.ipynb` | spare | `备件输机未联动清单/` | 在线表格追加 |
| 4 | 价格通知书商折识别 | `mass_price_identify_business_discount.ipynb` | mass | `价格通知书_需商折/` | 在线表格追加 |

### 二、供应链事件跟踪（2 个 — MRP 收货驱动）

| # | 检查项 | Notebook | 输入 | 输出 | 飞书 |
|---|--------|----------|------|------|------|
| 5 | 工程设变价格跟踪 | `engineering_change_price_track.ipynb` | EBOM + MRP + mass | `工程设变价格跟踪/` | 群消息通知 |
| 6 | 专用工装到期剥离 | `tooling_amortization_expiry.ipynb` | 工装台账 + MRP + mass | `专用工装到期剥离/` | 群消息通知 |

### 三、交互式执行（2 个 — 不可 nbconvert 批量运行）

| # | 检查项 | Notebook | 说明 |
|---|--------|----------|------|
| 7 | 物流方式检查 | `mass_logistic_different.ipynb` | 含 `input()`，须在 VSCode 中逐 cell 执行 |
| 8 | 备件量产件价格差异 | `mass_spare_difference.ipynb` | 含 `input()`，须在 VSCode 中逐 cell 执行 |

## 目录结构

```
compliance_report/
├── CLAUDE.md                     # 本文件
├── README.md                     # 面向用户的项目文档
├── .gitignore
├── .claude/
│   ├── .mcp.json                 # MCP 服务器配置
│   └── skills/                   # 8 个 Skill 定义（优先于 notebook 实现）
├── data/                         # 只读输入（7 个 Excel）
├── notebooks/                    # 8 个分析 Notebook
├── output/                       # 输出结果（8 个子目录）
│   ├── 联动标识检查结果/
│   ├── 价格通知书_需商折/
│   ├── 备件输机(应联动)未联动清单/
│   ├── 备件高于同期量产件清单/
│   ├── 物流方式不一致/
│   ├── 工程设变价格跟踪/
│   └── 专用工装到期剥离/
└── reports/                      # PPT 双周报
```

## Invocable 命令（4 个）

| 命令 | 交互流程 |
|------|---------|
| `/备件输机(应联动)未联动检查` | 询问日期区间 → Python 执行 → 报告结果 → 询问是否追加至飞书在线表格 |
| `/价格通知书_需商折` | 同上 |
| `/工程设变价格跟踪` | 询问日期区间 → Python 执行 → 报告结果 → 询问是否发送飞书群消息 |
| `/专用工装到期剥离` | 全量扫描（无需日期参数）→ Python 执行 → 报告结果 → 询问是否发送飞书群消息 |

## nbconvert 批量执行（6 个）

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/94项需改为联动标识检查.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/160项量产件检查跟踪_mass.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/备件输机(应联动)_未联动检查.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/mass_price_identify_business_discount.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/engineering_change_price_track.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/tooling_amortization_expiry.ipynb
```

> ⚠️ `mass_logistic_different.ipynb` 和 `mass_spare_difference.ipynb` 含 `input()` 交互逻辑，nbconvert 不支持，必须在 VSCode 中逐 cell 运行。

## 通用数据处理范式

所有 Notebook 遵循同一套模式：

1. **构造联合键**：`零件号_供应商编码`，贯穿全部分析的匹配主键
2. **状态筛选**：备件表 → `["已审批", "审批中"]`；量产件表 → `["已发布", "已确认", "已审批"]`
3. **日期清洗**：取日期列前 10 字符去掉时间戳；若生效/失效日期跨年，将失效日调整为生效年 12 月 31 日
4. **分组去重**：按 key 分组，失效日降序 → 创建时间降序，每组取第一条（即最新有效记录）
5. **左连接比对**：key1（待查清单）LEFT JOIN key2（价格表），比较裸价或提取状态字段
6. **导出 Excel**：路径格式 `../output/<子目录>/<文件名>_YYYYMMDD.xlsx`

> Notebook 工作目录为 `notebooks/`，读取文件使用 `../data/`，写入文件使用 `../output/`。

## 约束

- 禁止生成额外的 `.py` 脚本或 `.txt` 中间文件
- 禁止对含 `input()` 的交互式 notebook 使用 nbconvert 批量执行
- 禁止修改交互式 notebook 中的 `input()` 核心逻辑
- `.claude/skills/*.md` 的优先级高于 notebook 实现，执行时以 skill 定义为准
- 所有飞书操作（表格追加、群消息发送）必须征得用户明确同意后方可执行
