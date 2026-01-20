<!-- Copyright (c) 2025, Williams.Wang. All rights reserved. Use restricted under LICENSE terms. -->

## Dividend Calendar Dataset

这个仓库用于维护**可复用的分红/分配日历数据**（目前覆盖 `SPY`、`GDX`），方便你的多个量化项目在需要做 **dividend adjustment / total return** 等处理时，能从一个统一来源获取分红事件。

### 你会得到什么

- **标准化 CSV**：每个资产一份 `*_dividend_calendar.csv`，字段（schema）跨资产一致，可选字段允许为空
- **SPY 原始数据转换工具**：将 `data/raw_data/SPY.csv`（TSV + 多行表头）转换为标准 CSV
- **基础测试**：保证 raw->csv 的解析条数与关键字段稳定

### 目录结构

```
data/
  SPY_dividend_calendar.csv        # SPY 标准化分红/分配日历（完整版 schema）
  GDX_dividend_calendar.csv        # GDX 标准化分红/分配日历（完整版 schema，缺失字段留空）
  raw_data/
    SPY.csv                        # SPY 原始导出（TSV）
  templates/
    dividend_calendar_template.csv # 新增资产时复制的 CSV 模板（只有 header）
docs/
  DATA_SCHEMA.md                   # 字段契约/填写约定（必读）
```

### 数据 schema（字段契约）

请以 `docs/DATA_SCHEMA.md` 为准。核心要点：

- **必填**：`symbol`, `ex_date`, `amount`
- **可选**：`record_date`, `payable_date`, `is_special`, `note`, `pct_change_from_prev`, `pct_change_from_prev_year`, `prior_12m_yield`
- 所有日期均为 ISO 格式：`YYYY-MM-DD`
- `pct_change_*` / `prior_12m_yield` **只存数值**（例如 `8.9`），不带 `%`

### 下游项目如何使用（读取 CSV）

最小 Python 读取示例（仅标准库 + pandas；你也可以用任意语言读 CSV）：

```python
from __future__ import annotations

from pathlib import Path

import pandas as pd

dataset_root = Path("path/to/dividend_calendar_dataset")

spy_dividends = pd.read_csv(dataset_root / "data" / "SPY_dividend_calendar.csv")
spy_dividends["ex_date"] = pd.to_datetime(spy_dividends["ex_date"])

# 典型用法：按 ex_date 对齐到价格序列，做 dividend adjustment
spy_dividends = spy_dividends.sort_values("ex_date")
cash_dividends = spy_dividends.loc[spy_dividends["amount"].astype(float) > 0, ["ex_date", "amount"]]
```

建议下游项目的最佳实践：

- **固定版本**：下游依赖应 pin 到 git tag / release（不要直接依赖 `main`），保证回测与生产可复现。
- **只用你需要的列**：做价格分红调整通常只需要 `ex_date` + `amount`；其余列用于对账/解释/扩展。

### 如何新增资产（手动维护）

步骤建议：

- 复制模板：`data/templates/dividend_calendar_template.csv`
- 命名为：`data/<SYMBOL>_dividend_calendar.csv`（例如 `data/QQQ_dividend_calendar.csv`）
- 逐行补全数据：至少填 `symbol, ex_date, amount`，其余字段可留空
- 保持按 `ex_date` 从新到旧或旧到新一致（推荐旧->新，便于增量追加）


### 维护约定（建议）

- **历史修正要可追溯**：尽量“追加”，如必须改历史行，建议在 commit message / release note 中说明原因。
- **跨资产统一 schema**：字段缺失留空，而不是为某个资产自定义新列（降低下游维护成本）。

### 数据features
'symbol,ex_date,record_date,payable_date,is_special,amount,note,pct_change_from_prev,pct_change_from_prev_year,prior_12m_yield
'

