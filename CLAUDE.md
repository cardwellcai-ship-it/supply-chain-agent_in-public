# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

EPS 备件与量产件价格合规检查项目。通过分析 `eps3_spare.xlsx`（备件输机价格）和 `eps3_mass.xlsx`（量产件输机价格）与 `备件_不联动清单_整改及跟踪项.xlsx` 的匹配关系，检查联动标识状态、价格差异、物流方式纠偏等合规事项。

## Data Files

| File | Description |
|------|-------------|
| `eps3_spare.xlsx` | EPS备件输机价格数据（含通知书状态、是否联动字段） |
| `eps3_mass.xlsx` | EPS量产件输机价格数据（含状态列） |
| `备件_不联动清单_整改及跟踪项.xlsx` | 备件不联动清单（含对策列、采购工程师、供应商） |

## Notebook Analysis Tasks

Each `.ipynb` is a self-contained analysis. Run in project root directory.

### 联动标识检查

- **`94项需改为联动标识检查.ipynb`** — 检查备件清单中"不联动→联动（已确定）"项与 eps3_spare 的匹配状态，标记是否联动
  ```bash
  jupyter nbconvert --to notebook --execute --inplace 94项需改为联动标识检查.ipynb
  ```
- **`160项量产件检查跟踪_mass.ipynb`** — 检查备件清单中"不联动→联动（推进中）"项与 eps3_mass 量产件最新价格对比
  ```bash
  jupyter nbconvert --to notebook --execute --inplace 160项量产件检查跟踪_mass.ipynb
  ```

### 价格差异分析

- **`mass_spare_difference 20250527.ipynb`** — 备件与量产件价格差异分析
- **`mass_price_difference_annual_cost_202602.ipynb`** — 年度成本差异分析
- **`mass_price_difference_colours_202602.ipynb`** — 按颜色分类的价格差异分析
- **`mass_price_difference_different_areas_202602.ipynb`** — 按区域的价格差异分析

### 其他

- **`mass_logistic_different_20251226.ipynb`** — PLS与EPS物流方式纠偏核对
- **`mass_price_identify_business discount_20260605.ipynb`** — 价格通知书需商折识别
- **`价格通知书_需商折/价格通知书_需商折.xlsx`** — 商折累计输出文件

## Core Logic Pattern

所有分析 notebook 遵循同一模式：
1. **Step 1** — 从清单文件筛选指定条件记录，构造 key = `零件号/备件号_供应商编码`
2. **Step 2** — 从输机价格文件筛选有效状态记录，构造相同格式 key
3. **Step 3** — 按 key 分组，调整跨年失效日期，按失效日期降序→创建时间降序取每组最新一条
4. **Step 4** — key1 左连接 key2，比较/提取目标字段，导出 Excel

## Output Directories

- `联动标识检查结果/` — 94项和160项的检查结果 Excel
- `价格通知书_需商折/` — 商折识别结果
- `备件高于同期量产件清单/` — 备件高于量产件的差异清单

## Tech Stack

- Python (pandas, openpyxl) for data processing
- Jupyter Notebook (.ipynb) for interactive analysis
- All analysis scripts run locally via `python -c` or `jupyter nbconvert --execute`
