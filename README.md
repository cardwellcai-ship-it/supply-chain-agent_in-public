# 合规点检报告系统

东风乘用车 EPS（电子采购系统）的价格合规检查工具箱。简单说就是：输入几张大 Excel，跑一通 Python 分析，找出备件和量产件之间价格不一致、联动标识不对、物流方式搞错、商折该剥没剥等等问题，然后输出 Excel 或者发飞书通知采购员。

## 能干什么

一共六大类检查，8 个分析脚本：

```
┌─────────────────────────────────────────┐
│          日常合规点检（每次双周报都要跑）      │
│  ① 联动标识检查（94 项 + 160 项）           │
│  ② 备件输机未联动                          │
│  ③ 商折识别                               │
│  ④ 物流方式检查                            │
├─────────────────────────────────────────┤
│          事件驱动检查（有情况才跑）           │
│  ⑤ 工程设变价格跟踪                        │
│  ⑥ 专用工装到期剥离                        │
└─────────────────────────────────────────┘
```

| # | 检查项 | 啥时候跑 | 结果通知方式 |
|---|--------|---------|------------|
| ① | 联动标识检查 | 双周 | 导出 Excel |
| ② | 价格差异分析 | 双周 | 飞书在线表格 |
| ③ | 物流方式检查 | 双周 | 导出 Excel |
| ④ | 商折识别 | 双周 | 飞书在线表格 |
| ⑤ | 工程设变价格跟踪 | 有设变收货时 | 飞书群消息 |
| ⑥ | 工装到期剥离 | 累计达量时 | 飞书群消息 |

## 数据怎么流转的

```
输入（data/）                    处理（notebooks/）           输出（output/）
───────────                     ────────────────           ────────────

eps3_spare（备件价格）────┐
eps3_mass（量产件价格）───┤
备件不联动清单 ──────────┼──→  ① ② ③ ④ 合规点检  ──→  联动标识检查结果/
PLS 物流纠偏清单 ────────┘                           价格通知书_需商折/
                                                     备件输机未联动清单/
EBOM 设变表 ────────────┐                            备件高于量产件清单/
MRP 收货记录 ───────────┼──→  ⑤ ⑥ 事件跟踪    ──→  物流方式不一致/
工装分摊台账 ───────────┘                            工程设变价格跟踪/
                                                     专用工装到期剥离/
                                                             │
                                              飞书在线表格 / 飞书群消息
```

## 怎么跑

### 方式一：VSCode 里跑（日常推荐）
打开对应 notebook，一个 cell 一个 cell 执行就行。物流检查、价格差异这两个必须这样跑，因为有交互输入。

### 方式二：命令行批量跑（适合自动化/定时任务）
```bash
cd compliance_report

# 下面 6 个可以批量跑
jupyter nbconvert --to notebook --execute --inplace notebooks/94项需改为联动标识检查.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/160项量产件检查跟踪_mass.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/备件输机(应联动)_未联动检查.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/mass_price_identify_business_discount.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/engineering_change_price_track.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/tooling_amortization_expiry.ipynb

# 下面 2 个别用命令行跑，里面有 input()，会卡住
# notebooks/mass_logistic_different.ipynb
# notebooks/mass_spare_difference.ipynb
```

### 方式三：对 Claude 说命令（AI 辅助）
直接在对话框里敲：

```
/备件输机(应联动)未联动检查     → 问日期 → 跑分析 → 问你要不要追加飞书表格
/价格通知书_需商折              → 同上
/工程设变价格跟踪               → 问日期 → 跑分析 → 问你要不要发飞书群消息
/专用工装到期剥离               → 全量扫描 → 跑分析 → 问你要不要发群消息
```

## 飞书都连了啥

| 检查项 | 发到哪 | 具体地址 |
|--------|--------|---------|
| 备件输机未联动 | 飞书在线表格 | `feishu.cn/sheets/YR84sG2n2hGo2ftfFAOcK6jBnCc` |
| 商折识别 | 飞书在线表格 | `feishu.cn/sheets/Wlv3shWSUh8EDttheo2cj7D1neh`（子表：商折_20250401） |
| 价格差异 | 飞书在线表格 | `feishu.cn/sheets/HN2sspGeJhKIs6tHrBBchwnBn2f`（工作表：清单） |
| 工程设变跟踪 | 飞书群消息 | 通过 `lark-cli messenger send` 发 |
| 工装到期剥离 | 飞书群消息 | 同上 |

## 跑这个项目需要什么

- Python 3.10+，装了 pandas 和 openpyxl
- Jupyter（能跑 notebook 和 nbconvert）
- lark-cli（飞书那部分，不连飞书的话不用装也行）
- Claude Code 配了 ppt-mcp 和 powerbi-microsoft 两个 MCP

## 规矩

- 不要生成额外的 `.py` 脚本、`.txt` 中间文件，保持项目干净
- 输入数据放 `data/`，跑出来的结果放 `output/`，PPT 放 `reports/`
- 飞书操作（追加表格、发群消息）都会先问你再执行，不会自己偷偷发
- 如果你想了解每个检查的具体业务逻辑，看 `.claude/skills/` 下面的 skill 定义文件就行
