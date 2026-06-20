# CLAUDE.md

> AI Agent instruction file — authoritative reference for Claude Code

## Project

Dongfeng EPS price compliance inspection system. Core data: eps3_spare.xlsx + eps3_mass.xlsx. 6 capabilities across 8 skills.

## Capabilities

### Compliance (4)
| # | Skill | Notebook | Input | Output | Feishu |
|---|-------|----------|-------|--------|--------|
| 1 | 94 linkage check | 94需改为联动标识检查.ipynb | spare+不联动清单 | 联动标识检查结果/ | - |
| 2 | 160 mass check | 160项量产件检查跟踪_mass.ipynb | mass+不联动清单 | 联动标识检查结果/ | - |
| 3 | spare unlinked | 备件输机(应联动)_未联动检查.ipynb | spare | 备件输机未联动清单/ | table |
| 4 | business discount | mass_price_identify_business_discount.ipynb | mass | 价格通知书_需商折/ | table |

### Supply Chain (2)
| 5 | ECO price tracking | engineering_change_price_track.ipynb | EBOM+MRP+mass | 工程设变价格跟踪/ | msg |
| 6 | tooling amortization | tooling_amortization_expiry.ipynb | 工装台账+MRP+mass | 专用工装到期剥离/ | msg |

### Interactive (2 — no nbconvert)
| 7 | logistics check | mass_logistic_different.ipynb | VSCode cell-by-cell |
| 8 | spare-mass price diff | mass_spare_difference.ipynb | VSCode cell-by-cell |

## Structure

compliance_report/
├── CLAUDE.md / README.md / .gitignore
├── .claude/ (.mcp.json, skills/ x8)
├── data/ (7 xlsx)
├── notebooks/ (8 ipynb)
├── output/ (8 subdirs)
└── reports/ (PPT)

## Invocable Commands (4)

/备件输机(应联动)未联动检查 → date -> py -> report -> ask feishu table
/价格通知书_需商折           → date -> py -> report -> ask feishu table
/工程设变价格跟踪             → date -> py -> report -> ask feishu msg
/专用工装到期剥离             → full scan -> py -> report -> ask feishu msg

## nbconvert (6)

jupyter nbconvert --to notebook --execute --inplace notebooks/94需改为联动标识检查.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/160项量产件检查跟踪_mass.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/备件输机(应联动)_未联动检查.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/mass_price_identify_business_discount.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/engineering_change_price_track.ipynb
jupyter nbconvert --to notebook --execute --inplace notebooks/tooling_amortization_expiry.ipynb

## Data Paradigm

1. key = partNo_supplierCode
2. status filter: spare->[已审批,审批中]; mass->[已发布,已确认,已审批]
3. date clean: str[0:10], cross-year adjust expiry to Dec31 of start year
4. group dedup: by key, sort expiry desc -> create_time desc, take first
5. left join compare
6. export: ../output/<dir>/<name>_YYYYMMDD.xlsx

## Constraints

- NO .py / .txt intermediates
- interactive nb (logistics/diff): NO nbconvert, NO modify input() logic
- skills/ definitions override notebook
- notebook cwd = notebooks/ so paths use ../data/ and ../output/
- feishu ops require explicit user consent
