# CLAUDE.md

> 这个文件是给你（Claude Code）看的，帮你快速搞清楚这个项目是干嘛的、怎么跑、有哪些坑要注意。

## 这是个什么项目

东风乘用车 EPS 价格合规点检工具。说白了就是用 Python（pandas）把几张 Excel 表交叉比对，找出合规问题，然后把结果导出 Excel 或者发到飞书上。

核心输入是两张超大的 Excel：
- `eps3_spare.xlsx`（备件输机价格，约 15 万行）
- `eps3_mass.xlsx`（量产件输机价格，约 17 万行）

## 都有哪些检查

一共 8 个 notebook，6 大类检查，分成三组：

### 日常合规点检（4 个 — 每次跑都要用的）

| # | 干嘛的 | Notebook | 数据来源 | 结果去哪 | 要不要发飞书 |
|---|--------|----------|----------|----------|------------|
| 1 | 94 项联动标识检查 | `94项需改为联动标识检查.ipynb` | spare + 不联动清单 | `联动标识检查结果/` | 不用 |
| 2 | 160 项量产件跟踪 | `160项量产件检查跟踪_mass.ipynb` | mass + 不联动清单 | `联动标识检查结果/` | 不用 |
| 3 | 备件输机未联动 | `备件输机(应联动)_未联动检查.ipynb` | spare | `备件输机未联动清单/` | 飞书表格 |
| 4 | 价格通知书商折 | `mass_price_identify_business_discount.ipynb` | mass | `价格通知书_需商折/` | 飞书表格 |

### 供应链事件跟踪（2 个 — 有设变/工装到期时才跑）

| # | 干嘛的 | Notebook | 数据来源 | 结果去哪 | 发飞书 |
|---|--------|----------|----------|----------|--------|
| 5 | 工程设变价格跟踪 | `engineering_change_price_track.ipynb` | EBOM表 + MRP收货 + mass | `工程设变价格跟踪/` | 群消息 |
| 6 | 工装到期剥离 | `tooling_amortization_expiry.ipynb` | 工装台账 + MRP收货 + mass | `专用工装到期剥离/` | 群消息 |

### 需要人工交互的（2 个 — 别用 nbconvert 跑，会卡住）

| # | 干嘛的 | Notebook | 怎么跑 |
|---|--------|----------|--------|
| 7 | 物流方式检查 | `mass_logistic_different.ipynb` | 在 VSCode 里逐 cell 跑（里面有个 `input()`） |
| 8 | 备件量产件价格差异 | `mass_spare_difference.ipynb` | 同上，也有 `input()` |

## 目录长这样

```
compliance_report/
├── CLAUDE.md                # 就是本文件
├── README.md                # 给人看的文档
├── .gitignore
├── .claude/
│   ├── .mcp.json            # MCP：ppt-mcp 和 powerbi
│   └── skills/              # 8 个 skill 定义，比 notebook 优先级高
├── data/                    # 所有输入 Excel（只读，别改）
├── notebooks/               # 8 个分析 notebook
├── output/                  # 所有输出结果
│   ├── 联动标识检查结果/
│   ├── 价格通知书_需商折/
│   ├── 备件输机(应联动)未联动清单/
│   ├── 备件高于同期量产件清单/
│   ├── 物流方式不一致/
│   ├── 工程设变价格跟踪/
│   └── 专用工装到期剥离/
└── reports/                 # PPT 双周报
```

## 用户可以用斜杠命令调用的（4 个）

| 命令 | 你跟用户的交互流程 |
|------|------------------|
| `/备件输机(应联动)未联动检查` | 问日期区间 → 跑 Python → 报结果 → 问要不要追加飞书表格 |
| `/价格通知书_需商折` | 同上 |
| `/工程设变价格跟踪` | 问日期区间 → 跑 Python → 报结果 → 问要不要发飞书群消息 |
| `/专用工装到期剥离` | **不用问日期**，全量扫描 → 跑 Python → 报结果 → 问要不要发飞书群消息 |

## 哪些可以 nbconvert 批量跑（6 个）

```bash
jupyter nbconvert --to notebook --execute --inplace notebooks/94项需改为联动标识检查.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/160项量产件检查跟踪_mass.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/备件输机(应联动)_未联动检查.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/mass_price_identify_business_discount.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/engineering_change_price_track.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/tooling_amortization_expiry.ipynb
```

> ⚠️ `mass_logistic_different.ipynb` 和 `mass_spare_difference.ipynb` 里面有 `input()`，nbconvert 跑不了，必须在 VSCode 里手动逐 cell 执行。

## 所有 notebook 的共通处理逻辑

1. **拼 key**：`零件号_供应商编码`，这是贯穿所有分析的联合主键
2. **筛状态**：备件表取 `["已审批", "审批中"]`，量产件表取 `["已发布", "已确认", "已审批"]`
3. **洗日期**：日期列取前 10 位去掉时间戳；跨年的记录把失效日调到生效年 12 月 31 日
4. **去重**：按 key 分组，失效日降序 → 创建时间降序，每组只取第一条（等于取最新有效的）
5. **左连接**：key1（待查清单）LEFT JOIN key2（价格表），比对裸价/联动状态
6. **导出**：路径格式 `../output/<子目录>/<文件名>_YYYYMMDD.xlsx`

> 注意：notebook 的工作目录是 `notebooks/`，所以读文件用 `../data/`，写文件用 `../output/`。

## 铁律（必须遵守）

- 🚫 **不准**生成额外的 `.py` 脚本或 `.txt` 中间文件
- 🚫 **不准**用 nbconvert 跑那两个有 `input()` 的 notebook
- 🚫 **不准**修改交互式 notebook 里的 `input()` 核心逻辑
- 📌 skill 定义文件（`.claude/skills/*.md`）的优先级比 notebook 实现高，以 skill 为准
- 📌 所有飞书操作（不管是追加表格还是发群消息）**必须先问用户**，同意了才能动手
