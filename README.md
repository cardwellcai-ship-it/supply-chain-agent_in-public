# README.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

EPS备件与量产件价格合规检查项目。通过交叉比对 `eps3_spare.xlsx`（备件输机数据）与 `eps3_mass.xlsx`（量产件输机价格数据），检查联动标识、价格差异、物流方式、商折识别等合规性问题。

## 数据文件

| 文件 | 说明 | 行数 |
|------|------|------|
| `eps3_spare.xlsx` | EPS备件输机价格数据（~40列：备件号、通知书状态、是否联动、备件裸价、量产件号、对应零件裸价、生效/失效日期、审批通过时间等） | ~157k |
| `eps3_mass.xlsx` | EPS量产件输机价格数据（~30列：状态、零件号、零件裸价、价格通知书号、供应商编码、生效/失效日期等） | ~173k |
| `备件_不联动清单_整改及跟踪项.xlsx` | 备件不联动整改清单（254行，含对策列："不联动→联动（已确定）"/"不联动→联动（推进中）"） | ~254 |
| `PLS和EPS物流方式纠偏清单_武汉&襄阳_数据不停变化新增直接更新.xlsx` | 物流方式纠偏对照表 | - |

## 6 个 Notebook（Skills）

| Skill | Notebook | 执行方式 | 输入参数 |
|-------|----------|----------|----------|
| 94项联动标识检查 | `94项需改为联动标识检查.ipynb` | `nbconvert --execute` | 无 |
| 160项量产件检查 | `160项量产件检查跟踪_mass.ipynb` | `nbconvert --execute` | 无 |
| 备件输机未联动检查 | `备件输机(应联动)_未联动检查.ipynb` | `nbconvert --execute` | 修改 START_DATE/END_DATE |
| 价格通知书_需商折 | `mass_price_identify_business discount_20260605.ipynb` | `nbconvert --execute` | 修改 START_DATE/END_DATE |
| 物流方式检查 | `mass_logistic_different_20251226.ipynb` | ⚠️ VSCode逐cell运行（含input()） | 交互式输入日期 |
| 价格差异分析 | `mass_spare_difference 20250527.ipynb` | ⚠️ VSCode逐cell运行（含input()） | 交互式输入日期 |

### 执行命令

```bash
# 支持 nbconvert 的（4个）
jupyter nbconvert --to notebook --execute --inplace "94项需改为联动标识检查.ipynb"
jupyter nbconvert --to notebook --execute --inplace "160项量产件检查跟踪_mass.ipynb"
jupyter nbconvert --to notebook --execute --inplace "备件输机(应联动)_未联动检查.ipynb"
jupyter nbconvert --to notebook --execute --inplace "mass_price_identify_business discount_20260605.ipynb"

# 含交互式输入的（2个）— 需在 VSCode 中逐 cell 运行
# mass_logistic_different_20251226.ipynb
# mass_spare_difference 20250527.ipynb
```

## 输出目录

| 目录 | 内容 |
|------|------|
| `联动标识检查结果/` | 94项检查结果 + 160项跟踪结果 |
| `价格通知书_需商折/` | 商折识别累计总表 + 日期快照 |
| `备件输机(应联动)未联动清单/` | 应联动未联动备件清单 |
| `备件高于同期量产件清单/` | 备件裸价高于量产件裸价的差异记录 |

## 核心逻辑模式

1. **构造 key**：`零件号/备件号_供应商编码`
2. **筛选状态**：eps3_spare 筛选 `通知书状态`（已审批/审批中），eps3_mass 筛选 `状态 in ["已发布","已确认","已审批"]`
3. **分组去重**：按 key 分组，若生效/失效日期跨年则调整失效日期至生效年最后一天；按失效日期降序→创建时间降序，取每组第一条
4. **左连接比对**：key1 左连接 key2，比较备件裸价 vs 量产件裸价
5. **导出 Excel**：输出到对应目录，文件名含运行日期

## 约束

- **不生成额外的 .py 脚本、.txt 中间文件**
- 交互式 notebook（物流检查、价格差异分析）不可通过 nbconvert 批量执行
- 通知书中含有 input() 的 notebook 不要修改其核心逻辑
- skill 定义文件在 `.claude/skills/` 目录下

## Skill 配置文件

`.claude/skills/*.md` 中定义了每个 skill 的详细逻辑（包括商折分类关键词表等），需与 notebook 保持一致。
