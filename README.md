Beauty & Personal Care E-commerce Analytics Capstone
End-to-end analytics project: relational SQL layer → Python wrangling & EDA → GenAI-powered insight narrator.
Anyone can reproduce every number by following the steps below.
---
Repository layout
```
├── README.md
├── requirements.txt
├── .gitignore
├── sql/
│   ├── schema.sql              # tables, PK/FK, NOT NULL, DEFAULT, indexes
│   ├── seed_data.sql           # documentation entry-point
│   ├── load_seed.py            # re-runnable CSV → SQLite (blanks → NULL)
│   └── reports.sql             # 9 business queries + exact output comments
├── data/
│   ├── customers.csv           # 45 rows
│   ├── products.csv            # 16 rows
│   └── orders.csv              # 180 rows
├── analysis/
│   ├── clean_and_eda.py        # full Part-2 pipeline (10 steps)
│   ├── visualize.py            # two required charts + bonus trend
│   ├── cleaned_orders.csv      # produced artefact
│   └── cleaning_summary.json   # produced artefact
├── visualizations/
│   ├── category_return_rate.png
│   ├── monthly_revenue_corrected.png
│   └── monthly_revenue_trend.png
└── narrator/
    ├── build_findings.py       # writes findings.json from code only
    ├── findings.json
    ├── generate_narrative.py   # Gemini SCR + offline fallback + accuracy check
    └── sample_output.txt
```
---
Part 1 – SQL Relational Layer & Reporting
Schema
```bash
sqlite3 beauty.db < sql/schema.sql
```
Creates `customers`, `products`, `orders` with correct types, `NOT NULL` / `DEFAULT`, primary keys, foreign keys. `discount_pct` and `rating` stay nullable.
Seed (re-runnable)
```bash
python3 sql/load_seed.py
```
Loads the three CSVs
Blank cells become `NULL` (never empty strings)
Verifies final counts: 45 / 16 / 180
Nine business reports
```bash
sqlite3 -header -column beauty.db < sql/reports.sql
```
Exact query output is pasted as a comment block above each query.
#	Report	Technique
1	Order totals (count, revenue, AOV)	`COALESCE`, `/100.0`
2	Missing ratings	`COUNT(*)` vs `COUNT(col)`
3	Zero-order customers	`LEFT JOIN` + `NOT IN` (match)
4	High-return cities (>20 %)	`GROUP BY` + `HAVING`
5	Top 5 spenders + ranks 3–5	`ORDER BY` + `LIMIT`/`OFFSET`
6	Revenue by category	Three-table `JOIN`
7	Names starting with “A”	`LIKE`
8	Distinct acquisition sources	`DISTINCT`
9	Loyalty tier (Gold/Silver)	`ALTER` + `UPDATE CASE`
---
Part 2 – Python Wrangling & EDA
```bash
pip install -r requirements.txt
python3 analysis/clean_and_eda.py
python3 analysis/visualize.py
```
Pipeline steps (order is critical)
Step	Action
1. Load and inspect	Load all three CSVs; confirm raw orders shape `(180, 9)`
2. Standardise casing	`payment_method` 7 raw values → `{UPI, Card, COD}`
3. Remove duplicates	Natural-key double submissions; print dropped IDs `O0176`–`O0180`
4. Impute missing values	`discount_pct` → 0; `rating` → median (3.0)
5. Merge and reconcile	Compute `order_value`; explain why cleaned total (₹97 358.30) differs from Part-1 SQL total (₹99 860.20)
6. IQR outliers	Flag quantity outliers (qty 25 & 30); do not drop
7. COD return hypothesis	Hypothesis stated → tested by payment method → verdict SUPPORTED (COD 44.44 % vs Card 14.71 % / UPI 18.87 %)
8. Multi-level segmentation	Return rate by payment × city_tier; highest-risk = COD + Tier-2 (54.55 %)
9. Correlation analysis	Matrix + labels (negligible / weak / moderate / strong)
10. Outlier-corrected trend	Monthly revenue with vs without outliers; true peak = 2026-03
Visualisations produced
`visualizations/category_return_rate.png` – return-rate bar chart
`visualizations/monthly_revenue_corrected.png` – with vs without outliers line chart
---
Part 3 – GenAI-powered Insight Narrator
```bash
python3 narrator/build_findings.py      # writes findings.json from code
python3 narrator/generate_narrative.py  # Gemini or offline fallback
```
Requirement	Implementation
findings.json	Written exclusively by `build_findings.py` from `cleaning_summary.json` – never hand-typed
SCR narrative function	`call_gemini()` – system instruction + prompt built from findings JSON
Parameters + error handling	`temperature=0`, `max_output_tokens=2048`, timeout 30 s, try/except → dict
Offline fallback	Rich template narrative when no API key / package / network
Numeric accuracy check	Confirms five key figures (175, 97358, 25.1, 2026-03, 54.55) appear in the text
Set `GEMINI_API_KEY` (or `GOOGLE_API_KEY`) to enable the live Gemini path; otherwise the offline fallback runs automatically and still passes the accuracy check.
---
Full reproduction sequence
```bash
# --- Part 1 ---
sqlite3 beauty.db < sql/schema.sql
python3 sql/load_seed.py
sqlite3 -header -column beauty.db < sql/reports.sql

# --- Part 2 ---
pip install -r requirements.txt
python3 analysis/clean_and_eda.py
python3 analysis/visualize.py

# --- Part 3 ---
python3 narrator/build_findings.py
python3 narrator/generate_narrative.py
# (optional) export GEMINI_API_KEY=your_key   then re-run generate_narrative.py
```
Every number in the narrative, charts and SQL comments can be regenerated from the source CSVs by running the sequence above.
---
Key verified results (preview)
Metric	Value
Raw / cleaned orders	180 → 175
Cleaned revenue	₹97 358.30
Overall return rate	25.14 %
COD return rate	44.44 % (hypothesis SUPPORTED)
Highest-risk segment	COD + Tier-2 (54.55 %)
True peak month (outliers removed)	2026-03 (₹20 318.90)
Quantity outliers flagged	O0011 (qty 25), O0098 (qty 30)
---
Capstone project – Beauty & Personal Care E-commerce Analytics
