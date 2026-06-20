# 合规点检报告系统

东风奕派 EPS（电子采购系统）价格合规检查工具集。通过交叉比对备件与量产件输机价格数据，自动识别联动标识异常、价格差异、物流方式偏差、商折遗漏、工程设变后价格未更新、工装分摊到期未剥离等问题。可导出 Excel、追加至飞书在线表格、发送飞书群消息，并支持借助 PowerPoint MCP 生成汇报幻灯片。

## 六大检查能力

```
┌──────────────────────────────────────────────┐
│              日常合规点检（双周例行）             │
│  ① 联动标识检查（94 项 + 160 项）               │
│  ② 备件输机未联动检查                          │
│  ③ 价格通知书商折识别                          │
│  ④ 物流方式一致性检查                          │
├──────────────────────────────────────────────┤
│              事件驱动检查（按需触发）             │
│  ⑤ 工程设变价格维护跟踪                        │
│  ⑥ 专用工装分摊到期剥离                        │
└──────────────────────────────────────────────┘
```

| # | 检查项 | 触发条件 | 通知方式 |
|---|--------|---------|---------|
| ① | 联动标识检查 ×2 | 双周 | 导出 Excel |
| ② | 价格差异分析 | 双周 | 飞书在线表格 |
| ③ | 物流方式检查 | 双周 | 导出 Excel |
| ④ | 商折识别 | 双周 | 飞书在线表格 |
| ⑤ | 工程设变价格跟踪 | 设变后 MRP 收货时 | 飞书群消息 |
| ⑥ | 工装到期剥离 | 累计供货达分摊数量时 | 飞书群消息 |

## 数据流

```
输入层 (data/)                        分析层 (notebooks/)               输出层 (output/)
───────────                          ─────────────────                ────────────

eps3_spare.xlsx ──────┐
eps3_mass.xlsx ───────┤
备件_不联动清单 ───────┼────→  ① ② ③ ④ 合规点检  ────→  联动标识检查结果/
PLS物流纠偏清单 ───────┘                                 价格通知书_需商折/
                                                         备件输机未联动清单/
EBOM_设变版本表 ──────┐                                  备件高于同期量产件清单/
MRP_收货记录表 ───────┼────→  ⑤ ⑥ 事件跟踪    ────→  物流方式不一致/
定点会_工装分摊台账 ───┘                                  工程设变价格跟踪/
                                                         专用工装到期剥离/
                                                                 │
                                                    飞书在线表格 · 群消息
```

## 目录结构

```
compliance_report/
├── README.md                     # 本文件
├── CLAUDE.md                     # AI Agent 指令
├── .gitignore
├── .claude/
│   ├── .mcp.json
│   └── skills/                   # 8 个 Skill 定义
├── data/                         # 输入数据（7 个 Excel，只读）
├── notebooks/                    # 分析 Notebook（8 个 .ipynb）
├── output/                       # 运行输出（8 个子目录）
└── reports/                      # 双周报 PPT
```

## 运行方式

### 方式一：VSCode 逐 cell 执行（日常推荐）
在 VSCode 中打开 notebook，逐 cell 运行。物流方式检查、价格差异分析必须以此方式执行。

### 方式二：nbconvert 命令行批量执行
```bash
cd compliance_report

# 以下 6 个支持批量执行
jupyter nbconvert --to notebook --execute --inplace notebooks/94项需改为联动标识检查.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/160项量产件检查跟踪_mass.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/备件输机(应联动)_未联动检查.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/mass_price_identify_business_discount.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/engineering_change_price_track.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/tooling_amortization_expiry.ipynb

# 以下 2 个不可批量执行（含交互式 input()）
# notebooks/mass_logistic_different.ipynb
# notebooks/mass_spare_difference.ipynb
```

### 方式三：Claude Code Skill（AI 辅助）
```bash
/备件输机(应联动)未联动检查     # 询问日期 → 分析 → 飞书表格追加
/价格通知书_需商折               # 询问日期 → 分析 → 飞书表格追加
/工程设变价格跟踪                # 询问日期 → 分析 → 飞书群消息
/专用工装到期剥离                # 全量扫描 → 分析 → 飞书群消息
```

## 飞书集成

| 检查项 | 目标 | 地址 |
|--------|------|------|
| 备件输机未联动 | 在线表格 | `feishu.cn/sheets/YR84sG2n2hGo2ftfFAOcK6jBnCc` |
| 商折识别 | 在线表格 | `feishu.cn/sheets/Wlv3shWSUh8EDttheo2cj7D1neh`（商折_20250401） |
| 价格差异 | 在线表格 | `feishu.cn/sheets/HN2sspGeJhKIs6tHrBBchwnBn2f`（清单） |
| 工程设变 / 工装到期 | 群消息 | `lark-cli messenger send` |

## 环境依赖

- Python 3.10+，pandas，openpyxl
- Jupyter Notebook / nbconvert
- lark-cli（飞书操作，可选）
- Claude Code MCP：ppt-mcp，powerbi-microsoft

## 项目约束

- 不生成额外的 `.py` 脚本或 `.txt` 中间文件
- 输入数据统一存放于 `data/`，输出结果统一存放于 `output/`，汇报材料存放于 `reports/`
- `.claude/skills/*.md` 为各检查项的权威定义，执行时以此为准
- 所有飞书写操作必须经用户确认后方可执行
