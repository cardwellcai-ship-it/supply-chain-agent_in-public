# 供应链数据分析Agent

通过交叉比对备件与量产件输机价格数据，自动识别联动标识异常、价格差异、物流方式偏差、商折遗漏、工程设变后价格未更新、工装分摊到期未剥离等问题。可直接导出分析结果，借助飞书CLI联动飞书在线资源(多维表格，飞书任务等)、发送飞书群消息，并支持借助 PowerPoint MCP 生成汇报幻灯片report，以及Power bi MCP 数据建模与可视化。

IDE：VScode

Vibe coding tool：claude code，codex

LLM:Deepseek v4-flash/pro

## 六大业务痛点

| # | 检查项 | 业务痛点 |
|---|--------|---------|
| 1 | 物流方式检查 | SAP价格系统与PLS物流系统均需维护零件的物流方式，如不一致会造成费用额外支付 |
| 2 | 零件及备件的价格差异分析 | SAP中备件价格与量产件价格分别维护，同一时间点如价格差异造成多结算损失 |
| 3 | 工程设变价格跟踪 | 工程设变引起零部件定义变化，造成价格变化后；新状态零部件到货后未及时维护价格也可造成多结算损失 |
| 4 | 工装到期剥离 | 项目开发阶段零件的特殊专用工装分摊金额，在供货量达到约定数量时，未及时剥离(降低)零件价格引起多结算损失 |
| 5 | 价格通知书商折识别 | 新增价格输机时 |
| 6 | 备件联动标识 | 建立备件与量产件联动机制后(量产件维护价格后，自动生成同期备件价格)，如备件联动标识维护错误，此机制无法生效|

## 业务痛点触发条件、通知方式

| # | 检查项 | 触发条件 | 通知方式 |
|---|--------|---------|---------|
| 1 | 物流方式检查 | 每周 | 导出 Excel |
| 2 | 零件及备件的价格差异分析 | 按需（事件触发） | 飞书在线表格 |
| 3 | 工程设变价格跟踪 | 设变后 MRP 收货时 | 飞书群消息 |
| 4 | 工装到期剥离 | 累计供货达分摊数量时 | 飞书群消息 |
| 5 | 价格通知书商折识别 | 新增价格输机时 | 飞书群消息 |
| 6 | 备件输机未联动检查 | 每周 | 飞书在线表格 |

## 数据流

```
输入层 (data/)                 分析层 (notebooks/)        输出层 (output/)            信息输出 (notice/) 
───────────                    ────────────────          ────────────

eps3_spare      ──────┐
eps3_mass      ───────┤
备件_不联动清单 ───────┼────→  1/2/3/6合规点检  ────→  联动标识检查结果/        ────→    飞书在线变更更新
PLS物流纠偏清单 ───────┘                              价格通知书_需商折/                 飞书消息通则
                                                     备件输机未联动清单/
                                                     备件高于同期量产件清单/
                                                     物流方式不一致/

EBOM_设变版本表 ──────┐                             
MRP_收货记录表 ───────┼────→  4/5事件跟踪    ────→   工程设变价格跟踪/         ────→     飞书消息通则
定点会_工装分摊台账 ───┘                             专用工装到期剥离/
                                                    

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
├── data/                         # 输入数据（7 个输入信息，只读）
├── notebooks/                    # 分析 Notebook（8 个 .ipynb分析scripts）
├── output/                       # 运行输出（7 个业务子目录）
└── reports/                      # 周报
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
notebooks/mass_logistic_different.ipynb
notebooks/mass_spare_difference.ipynb

```

### 方式三：Claude Code Skill（AI Agent自然语言启动）
```bash
/备件输机(应联动)未联动检查       # 询问日期 → 分析 → 飞书表格追加
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
- lark-cli（飞书操作）
- Claude Code MCP：ppt-mcp，powerbi-microsoft

## 项目约束

- 不生成额外的 `.py` 脚本或 `.txt` 中间文件
- 输入数据统一存放于 `data/`，输出结果统一存放于 `output/`
- `.claude/skills/*.md` 为各检查项的权威定义，执行时以此为准
- 所有飞书信操作必须经用户确认后方可执行
