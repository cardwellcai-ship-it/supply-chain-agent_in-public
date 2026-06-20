# Compliance Report System

Dongfeng EPS price compliance toolkit. 6 capabilities across 8 analysis notebooks.

## Capabilities



## Structure

compliance_report/
├── README.md / CLAUDE.md / .gitignore
├── .claude/ (skills/, .mcp.json)
├── data/ (7 input xlsx)
├── notebooks/ (8 ipynb)
├── output/ (8 result dirs)
└── reports/ (PPT)

## Run

| Method | Use |
|--------|-----|
| VSCode cell-by-cell | interactive notebooks (logistics, price diff) |
| nbconvert --execute --inplace | 6 non-interactive notebooks |
| Claude Code Skill | /command for 4 invocable skills |

### Skill Commands

/备件输机(应联动)未联动检查
/价格通知书_需商折
/工程设变价格跟踪
/专用工装到期剥离

## Feishu Integration

| Check | Target |
|-------|--------|
| spare unlinked | table: feishu.cn/sheets/YR84sG2n2hGo2ftfFAOcK6jBnCc |
| business discount | table: feishu.cn/sheets/Wlv3shWSUh8EDttheo2cj7D1neh |
| price diff | table: feishu.cn/sheets/HN2sspGeJhKIs6tHrBBchwnBn2f |
| ECO / Tooling | group message via lark-cli |

## Constraints

- No .py/.txt intermediates
- data/ in, output/ out, reports/ ppt
- .claude/skills/*.md is authoritative for AI execution
