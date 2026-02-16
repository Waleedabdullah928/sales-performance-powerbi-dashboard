# DAX Measures — Sales Performance & Pipeline Intelligence

> 

---

## 1) Core Counts & Revenue

### Deal Count
```DAX
Deal Count =
COUNTROWS(sales_pipeline)
```

### Total Revenue
```DAX
Total Revenue =
SUM(sales_pipeline[Product Value])
```

### Won Revenue
```DAX
Won Revenue =
CALCULATE(
    SUM(sales_pipeline[Product Value]),
    sales_pipeline[deal_stage] = "Won"
)
```

### Lost Revenue
```DAX
Lost Revenue =
CALCULATE(
    SUM(sales_pipeline[Product Value]),
    sales_pipeline[deal_stage] = "Lost"
)
```

---

## 2) Win Rate

### Won Deals
```DAX
Won Deals =
CALCULATE(
    COUNTROWS(sales_pipeline),
    sales_pipeline[deal_stage] = "Won"
)
```

### Total Deals
```DAX
Total Deals =
COUNTROWS(sales_pipeline)
```

### Win Rate %
```DAX
Win Rate % =
DIVIDE([Won Deals], [Total Deals])
```

> Format: **Percentage** (1 decimal)

---

## 3) Pipeline (Open Deals)

### Open Pipeline Value
```DAX
Open Pipeline Value =
CALCULATE(
    SUM(sales_pipeline[Product Value]),
    ISBLANK(sales_pipeline[close_date])
)
```

### Open Deals
```DAX
Open Deals =
CALCULATE(
    COUNTROWS(sales_pipeline),
    ISBLANK(sales_pipeline[close_date])
)
```

---

## 4) Aging (Open Deals Only)

### Open Deal Age (Days) — Calculated Column
```DAX
Open Deal Age (Days) =
IF(
    ISBLANK(sales_pipeline[close_date]),
    DATEDIFF(
        sales_pipeline[engage_date],
        TODAY(),
        DAY
    )
)
```

### Age Bucket — Calculated Column
```DAX
Age Bucket =
SWITCH(
    TRUE(),
    sales_pipeline[Open Deal Age (Days)] <= 30, "0–30 Days",
    sales_pipeline[Open Deal Age (Days)] <= 60, "31–60 Days",
    sales_pipeline[Open Deal Age (Days)] <= 90, "61–90 Days",
    "90+ Days"
)
```

### % of Open Pipeline (by Age Bucket)
```DAX
% of Open Pipeline =
DIVIDE(
    [Open Pipeline Value],
    CALCULATE([Open Pipeline Value], ALL(sales_pipeline[Age Bucket]))
)
```

---

## 5) Deal Size

### Avg Deal Size
```DAX
Avg Deal Size =
AVERAGE(sales_pipeline[Product Value])
```

### Avg Won Deal Size
```DAX
Avg Won Deal Size =
CALCULATE(
    AVERAGE(sales_pipeline[Product Value]),
    sales_pipeline[deal_stage] = "Won"
)
```

---

## 6) Stage Ordering (Recommended)

### Stage Table (Dimension) — New Table
```DAX
Stage Table =
DATATABLE(
    "deal_stage", STRING,
    "Stage Order", INTEGER,
    {
        {"Prospecting", 1},
        {"Engaging", 2},
        {"Negotiation", 3},
        {"Won", 4},
        {"Lost", 5}
    }
)
```

**Modeling steps**
- Create relationship: `Stage Table[deal_stage]` → `sales_pipeline[deal_stage]`
- Sort: Select `Stage Table[deal_stage]` → **Sort by column** → `Stage Order`

---

## 7) Geo (Location)

### Open Pipeline (Geo)
```DAX
Open Pipeline (Geo) =
[Open Pipeline Value]
```

---

## 8) Agent Performance Matrix (Scatter)

Use these measures in your scatter chart:
- **X Axis**: `[Win Rate %]`
- **Y Axis**: `[Won Revenue]`
- **Size**: `[Avg Deal Size]`
- **Details**: `sales_pipeline[sales_agent]`

---

## 9) Forecast Helpers (Optional)

### Monthly Won Revenue
```DAX
Monthly Won Revenue =
CALCULATE(
    SUM(sales_pipeline[Product Value]),
    sales_pipeline[deal_stage] = "Won"
)
```

> Tip: Use a proper Date table + month axis for best results.

---

## 10) Utility (Optional)

### Top 10 Agents (Filter)
```DAX
Top 10 Agents (Filter) =
VAR RankAgent =
    RANKX(
        ALL(sales_pipeline[sales_agent]),
        [Won Revenue],
        ,
        DESC
    )
RETURN
    IF(RankAgent <= 10, 1, 0)
```

Use as a visual filter: `Top 10 Agents (Filter) = 1`.

---

### Author
Waleed Abdullah — Power BI / DAX Portfolio Measures
