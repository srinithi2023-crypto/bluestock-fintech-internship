[India Village Geographical Data.md](https://github.com/user-attachments/files/30297668/India.Village.Geographical.Data.md)[Upload# B100 Intelligence — Fundamental Analysis for Stock Market Listed Companies

> A financial intelligence system for the Nifty 100 — India's top 100 publicly listed
> companies — combining a dimensional data warehouse, seven Power BI dashboards, an ML
> scoring engine, and a Django web application with a partner-facing REST API.

| | |
|---|---|
| **Start date** | 01 May 2026 |
| **Status** | Active — 95% complete |
| **Priority** | High |
| **Team size** | 7 |
| **Effort** | 8 weeks × 5 days × 3 hours ≈ 120 hours |

---

## 1. Overview

The project has three parallel output streams:

| Stream | Focus | Share of effort |
|--------|-------|-----------------|
| **A — Power BI Analytics** | Seven production-grade dashboards usable by any investor or analyst without writing SQL | 40% |
| **B — Data Engineering & Python Analytics** | ETL pipelines, PostgreSQL star schema, ML health scores | 35% |
| **C — Django Web App & API** | Public website plus a channel-partner REST API | 25% |

Weeks 1–5 are entirely data work. Django only starts in Week 6 — the correct order for
an analytics-led project.

---

## 2. Source data

A MariaDB export (`scriptticker.sql`) containing roughly 12 years of financials per
company across seven tables.

| Table | Contents |
|-------|----------|
| `companies` | Master record: name, logo, website, NSE/BSE links, ROCE, ROE, face value, book value |
| `analysis` | Growth metrics for 10Y / 5Y / 3Y / TTM — compounded sales growth, profit growth, stock CAGR, ROE |
| `balancesheet` | Equity capital, reserves, borrowings, liabilities, fixed assets, total assets |
| `profitandloss` | Sales, expenses, operating profit, OPM%, interest, depreciation, net profit, EPS, dividend |
| `cashflow` | Operating, investing, financing activity and net cash flow |
| `prosandcons` | Pre-curated pros and cons text per company |
| `documents` | BSE annual report PDF links per company per year |

**Key quirks handled in ETL:**

- `company_id` (e.g. `TCS`, `HDFCBANK`, `INFY`) is a **string primary key** used for every join.
- Year values are inconsistent across tables — `Mar 2024`, `Sep 2024`, `Mar-24`, `TTM`.
- `analysis` stores metrics as strings like `"10 Years: 11%"` that must be parsed into
  a period label and a numeric value.
- Nulls arrive as the strings `'NULL'` / `'Null'`.

### Sector coverage

IT (TCS, INFY, WIPRO) · Banking (HDFCBANK, AXISBANK, BANKBARODA) · Energy (ADANIGREEN,
ADANIPOWER, ADANIENSOL, ATGL) · Pharma · FMCG · Cement (AMBUJACEM) · Healthcare
(APOLLOHOSP) · Paints (ASIANPAINT) · Insurance (SBILIFE) · Auto (BAJAJ-AUTO) ·
Finance (BAJFINANCE, BAJAJFINSV) · and more.

---

## 3. Technology stack

| Category | Tool | Purpose |
|----------|------|---------|
| Primary BI | Power BI Desktop + Service | All 7 dashboards |
| Measures | DAX | KPIs, calculated measures, dynamic filtering |
| Transformation (BI) | Power Query (M) | In-model shaping |
| ETL | Python 3.11 · pandas · SQLAlchemy | Cleaning, transformation, loading |
| Warehouse | PostgreSQL 15 | Star-schema warehouse Power BI connects to |
| Source DB | MySQL / MariaDB dump | Raw data |
| Analytics | pandas, numpy, scipy, scikit-learn, matplotlib, seaborn | Analysis and ML scoring |
| Notebooks | Jupyter | EDA and modelling |
| Web | Django 4.2 + Django REST Framework | Website + partner API |
| Background jobs | Celery + Redis | Scheduled refresh and scoring |
| Web charts | Chart.js | Company detail page visuals |
| API docs | drf-spectacular (Swagger/OpenAPI) | Partner documentation |
| VCS | Git + GitHub | Code, notebooks, PBIX files |
| Containers | Docker + Docker Compose | Local PostgreSQL, Redis, Django, Celery |

---

## 4. Data warehouse design

Power BI performs best against a star schema, so the raw operational tables are
restructured into dimensions and facts.

### Dimension tables

| Table | Primary key | Columns |
|-------|-------------|---------|
| `dim_company` | `symbol` VARCHAR(20) | symbol, company_name, sector, sub_sector, company_logo, website, nse_url, bse_url, face_value, book_value, about_company |
| `dim_year` | `year_id` INT | year_id, year_label (`'Mar 2024'`), fiscal_year, quarter, is_ttm, is_half_year, sort_order |
| `dim_sector` | `sector_id` INT | sector_id, sector_name, sector_code, description |
| `dim_health_label` | `label_id` INT | label_id, label_name (EXCELLENT/GOOD/AVERAGE/WEAK/POOR), min_score, max_score, color_hex |

### Fact tables

| Table | Grain | Key columns |
|-------|-------|-------------|
| `fact_profit_loss` | 1 row per company per year | sales, expenses, operating_profit, opm_pct, other_income, interest, depreciation, profit_before_tax, tax_pct, net_profit, eps, dividend_payout_pct |
| `fact_balance_sheet` | 1 row per company per year | equity_capital, reserves, borrowings, other_liabilities, total_liabilities, fixed_assets, cwip, investments, other_assets, total_assets, debt_to_equity |
| `fact_cash_flow` | 1 row per company per year | operating_activity, investing_activity, financing_activity, net_cash_flow, free_cash_flow |
| `fact_analysis` | 1 row per company per period | period_label (10Y/5Y/3Y/TTM), compounded_sales_growth_pct, compounded_profit_growth_pct, stock_price_cagr_pct, roe_pct |
| `fact_ml_scores` | 1 row per company per run | computed_at, overall_score, profitability_score, growth_score, leverage_score, cashflow_score, dividend_score, trend_score, health_label |
| `fact_pros_cons` | 1 row per insight per company | is_pro, category, text, source (MANUAL/ML), confidence, generated_at |

### Derived metrics computed during ETL

| Metric | Formula | Target table |
|--------|---------|--------------|
| `debt_to_equity` | `borrowings / (equity_capital + reserves)` | fact_balance_sheet |
| `net_profit_margin_pct` | `(net_profit / sales) * 100` | fact_profit_loss |
| `expense_ratio_pct` | `(expenses / sales) * 100` | fact_profit_loss |
| `interest_coverage` | `operating_profit / interest` | fact_profit_loss |
| `free_cash_flow` | `operating_activity + investing_activity` | fact_cash_flow |
| `cash_conversion_ratio` | `operating_activity / net_profit` | fact_cash_flow ⋈ fact_profit_loss |
| `asset_turnover` | `sales / total_assets` | fact_profit_loss ⋈ fact_balance_sheet |
| `return_on_assets` | `(net_profit / total_assets) * 100` | fact_profit_loss ⋈ fact_balance_sheet |
| `equity_ratio` | `(equity_capital + reserves) / total_assets` | fact_balance_sheet |
| `book_value_per_share` | `(equity_capital + reserves) / shares_outstanding` | fact_balance_sheet |

---

## 5. ETL pipeline

Three standalone modules under `etl/`.

### `etl/01_extract_from_mysql.py`
Parses `INSERT INTO` statements from the SQL dump with regex and writes each of the
seven tables to `data/raw/*.csv`. Handles escaped quotes, special characters and NULL
literals. Prints row count and column list per table for verification.

### `etl/02_clean_and_transform.py`
Standardises raw CSVs into `data/clean/`:

- **Year standardisation** — `Mar-24 → Mar 2024`, `TTM → TTM`; derives `fiscal_year` (int) and `sort_order` (int)
- **Analysis parsing** — `"10 Years: 11%"` → `period = '10Y'`, `value_pct = 11.0`
- **Null handling** — string `'NULL'`/`'Null'` → `NaN`
- **Name cleaning** — strip trailing whitespace and `\r\n` from `company_name`
- **Sector classification** — all 100 companies mapped to IT, Banking, NBFC, Insurance,
  Energy, Power, Ports, Cement, Healthcare, Auto, Paint, Consumer Goods, Holding Company;
  saved as `data/sector_mapping.csv`
- **Computed columns** — every derived metric from §4

### `etl/03_load_to_warehouse.py`
Loads into PostgreSQL in dependency order: dimensions first (company, year, sector),
then facts with FK references. Uses `ON CONFLICT DO UPDATE` so the script is
**idempotent** and safe to re-run. Prints row counts before and after, then runs the
data-quality gate — all checks must pass before downstream work proceeds.

---

## 6. Power BI dashboards

Seven PBIX files in `powerbi/`. Each has a defined audience, page set and mandatory visuals.

### Connection and model

Get Data → PostgreSQL Database → `localhost:5432` / `bluestock_dw` → import all `dim_*`
and `fact_*` tables. Verify types and null counts in Power Query. Credentials are never
hard-coded — use environment variables or a Power BI Gateway.

**Relationships (all Many-to-One):**

| From | Column | To | Column |
|------|--------|----|--------|
| fact_profit_loss | symbol / year_id | dim_company / dim_year | symbol / year_id |
| fact_balance_sheet | symbol / year_id | dim_company / dim_year | symbol / year_id |
| fact_cash_flow | symbol / year_id | dim_company / dim_year | symbol / year_id |
| fact_analysis | symbol | dim_company | symbol |
| fact_ml_scores | symbol | dim_company | symbol |
| fact_pros_cons | symbol | dim_company | symbol |
| dim_company | sector | dim_sector | sector_name |

### Dashboard summary

| # | File | Audience | Pages |
|---|------|----------|-------|
| 1 | `01_executive_overview.pbix` | Fund managers, CXOs needing a 30-second snapshot | 3 |
| 2 | `02_company_deep_dive.pbix` | Individual investors, research analysts | 4 |
| 3 | `03_sector_comparison.pbix` | Analysts comparing IT vs Banking vs Energy etc. | 3 |
| 4 | `04_health_scorecard.pbix` | Risk analysts, portfolio managers | 2 |
| 5 | `05_growth_analytics.pbix` | Growth investors tracking acceleration | 3 |
| 6 | `06_debt_leverage.pbix` | Credit analysts, risk managers, value investors | 2 |
| 7 | `07_dividend_returns.pbix` | Income investors, dividend funds | 2 |

<details>
<summary><strong>Dashboard 1 — Executive Market Overview</strong></summary>

**Page 1.1 — Market Snapshot**

| Visual | Type | Configuration |
|--------|------|---------------|
| Total Companies | Card | `COUNT(dim_company[symbol])` |
| Average ROE | Card | `AVERAGE(fact_profit_loss[roe_pct_3y])` |
| Excellent Health Count | Card | `COUNTROWS(FILTER(fact_ml_scores, score >= 85))` |
| Weak/Poor Health Count | Card | `COUNTROWS(FILTER(fact_ml_scores, score < 50))` |
| Sector Distribution | Donut | Company count by sector |
| Health Label Distribution | Bar | Company count by health label |
| Top 10 by ROE | Horizontal bar | Company vs `[ROE Last Year]` |
| Bottom 10 by Growth | Horizontal bar | Company vs `[3Y Sales CAGR]` |
| Sector Avg OPM% | Matrix heatmap | Sector × Year, `AVG(opm_pct)` |

**Page 1.2 — Sector Performance**
Sector revenue trend (clustered bar), profitability scatter (X = 3Y sales growth,
Y = 3Y ROE, size = revenue, colour = sector), sector scorecard matrix, treemap
(area = revenue, colour = health score), best company per sector.

**Page 1.3 — YoY Growth Tracker**
Aggregate Nifty 100 revenue line, total net profit area chart, growth-rate histogram,
year slicer (2013 → latest), multi-select company slicer.
</details>

<details>
<summary><strong>Dashboard 2 — Company Deep Dive</strong> (the most important dashboard)</summary>

A **Company Selector slicer sits at the top of every page** and drives all visuals.

**Page 2.1 — Financial Summary** — company name card, health score gauge (0–35 red,
35–50 orange, 50–70 yellow, 70–85 light green, 85–100 green), health label card,
revenue-vs-profit combo (12 years), OPM% trend with sector-average reference line,
EPS bar chart, per-year metrics table, four CAGR cards (10Y/5Y/3Y/TTM).

**Page 2.2 — Balance Sheet Health** — D/E trend with reference line at 1.0, 100%
stacked asset composition, borrowings (red) vs reserves (green) area chart, total
asset growth, 8-year balance sheet matrix, equity ratio line.

**Page 2.3 — Cash Flow Analysis** — waterfall (operating green, investing blue,
financing orange), free cash flow trend with positive/negative colouring, cash
conversion ratio with reference at 1.0, operating cash flow vs net profit dual-line
(divergence signals earnings-quality risk), full cash flow table.

**Page 2.4 — Growth & Returns** — YoY sales and profit growth bars (green/red),
ROE vs sector average band, dividend history (bar = payout %, line = EPS), CAGR radar
across 10Y/5Y/3Y, return metrics summary (ROE, ROCE, ROA).
</details>

<details>
<summary><strong>Dashboards 3–7</strong></summary>

**3 — Sector Comparison Analyzer**
Page 3.1 sector-vs-sector (multi-select 2–4 sectors, revenue/profitability/leverage
comparison, scorecard matrix). Page 3.2 companies within a sector (single-select
dropdown, company list, revenue and health rankings, 3-year OPM% comparison).
Page 3.3 sector trends over time (year range slicer, combined revenue growth,
margin evolution, health-label shift stacked bar).

**4 — Financial Health Scorecard**
Page 4.1 leaderboard (rank, company, sector, score, label, OPM%, D/E, 3Y growth, ROE),
score histogram, score-by-sector box plot, "Companies Needing Attention" filtered to
WEAK/POOR with key issues drawn from `fact_pros_cons`. Page 4.2 breakdown — 6-axis
radar (Profitability, Growth, Leverage, Cash Flow, Dividend, Trend), score trend across
runs, pros/cons tables, peer comparison against top 3 and bottom 3 in sector.

**5 — Growth & Valuation Analytics**
Page 5.1 revenue and profit growth (YoY waterfall, growth scatter with quadrant lines
at 10%/10%, consistent-growers DAX flag, CAGR acceleration comparison). Page 5.2
margin evolution (OPM% heat map: red <10%, yellow 10–20%, green >20%; net margin trend,
expense ratio, margin improvement leaders, interest coverage with danger lines at 2.0
and 3.0). Page 5.3 EPS and earnings quality (EPS 3Y CAGR ranking, EPS vs cash
conversion scatter, earnings consistency matrix).

**6 — Debt & Leverage Monitor**
Page 6.1 leverage snapshot (D/E heat map: red >2, yellow 1–2, green <1; borrowings
trend for the 10 most leveraged, D/E vs interest coverage scatter with danger quadrant,
debt-free companies table, biggest YoY D/E increases, interest coverage ranking).
Page 6.2 debt trajectory (borrowings journey, reserves vs borrowings crossover,
financing activity as a repayment signal, sector average reference line).

**7 — Dividend & Shareholder Returns**
Page 7.1 dividend analysis (top 20 payout ranking, consistent payers DAX flag, payout
vs EPS growth scatter, per-company payout trend). Page 7.2 shareholder value (10-year
EPS compounding, ROE last year vs 3Y vs 10Y, stock price CAGR table, 4-axis value
creation radar).
</details>

### Formatting standards

Applied across all seven files so they read as one product.

| Element | Standard |
|---------|----------|
| Primary colour | `#1F4E79` dark blue |
| Secondary colour | `#2E75B6` medium blue |
| Positive / good | `#2ECC71` green |
| Warning / medium | `#F39C12` orange |
| Negative / bad | `#E74C3C` red |
| Background | `#F8F9FA` off-white |
| Card background | `#FFFFFF` |
| Font | Segoe UI — 12 body, 20 card values, 14 axis labels |
| Number cards | INR symbol with `Cr` suffix |
| Title format | `COMPANY NAME — Dashboard Title` (top left, dark blue, bold, 18) |
| Footer | `Data as of: [date]. For educational purposes only. Not financial advice.` |

---

## 7. Python analytics & ML engine

Six Jupyter notebooks under `notebooks/`.

| Notebook | Purpose |
|----------|---------|
| `01_eda.ipynb` | Null heatmap, distributions, correlation matrix, outlier analysis (IQR + Z-score), written insight summary |
| `02_health_scoring.ipynb` | 6-dimension composite scoring across all 100 companies |
| `03_anomaly_detection.ipynb` | Z-score and Isolation Forest, cross-compared |
| `04_sector_clustering.ipynb` | K-Means (K=5) with PCA visualisation and written cluster descriptions |
| `05_peer_comparison.ipynb` | Cosine-similarity peer matrix → `peer_mapping.csv` |
| `06_trend_forecasting.ipynb` | ARIMA 1-year revenue forecasts for the top 20 companies |

**Sanity checks used during scoring:** does TCS score above Wipro? Does ADANIPOWER
score lower on leverage? Scores that fail business intuition indicate a formula problem,
not a data problem.

Scoring and anomaly functions are wrapped as **Celery tasks** with a Celery Beat
schedule, writing results back into `fact_ml_scores` and `fact_pros_cons`.

---

## 8. Django web application & API

### App structure

| App | Purpose |
|-----|---------|
| `companies` | Models, services and views for company financial data |
| `ml_engine` | Scoring tasks, models for scores, anomalies, trends |
| `api_management` | Partner API key management, rate limiting, usage logs |
| `dashboard` | User-facing HTML template pages |
| `accounts` | JWT auth, admin user management, permissions |

Django points at the **same PostgreSQL database** as the warehouse — no data duplication.
Models are bootstrapped with `inspectdb` against the existing tables.

### Public pages

| Route | Content |
|-------|---------|
| `/` | Search bar, featured companies, sector cards, AI insights ticker |
| `/companies/` | Filterable table — sector, health label, debt level; sortable by ROE, score, growth |
| `/company/{symbol}/` | Company detail: info, health gauge, 8 Chart.js charts, pros/cons, annual reports |
| `/compare/` | Side-by-side comparison of 2–4 companies |
| `/screener/` | Multi-filter form — min ROE, max D/E, sector, health label, min sales growth |
| `/sector/{name}/` | All companies in a sector with a combined performance chart |

### The 8 required charts (company detail page)

| # | Chart | Type | Data |
|---|-------|------|------|
| 1 | Revenue & Profit Trend | Grouped bar + line | Sales, net profit (bars), OPM% (line) — 12 years |
| 2 | Balance Sheet Composition | Stacked bar | Equity+Reserves (green), Borrowings (red), Other Liabilities (orange) |
| 3 | Cash Flow Waterfall | 3-series bar | Operating (green), Investing (blue), Financing (orange) |
| 4 | EPS & Dividend History | Dual line | EPS and dividend payout % |
| 5 | Debt vs Equity | Area | Borrowings (red) vs Reserves (green) |
| 6 | CAGR Radar | Spider | Sales, Profit, Stock CAGR, ROE — overlaid 10Y/5Y/3Y |
| 7 | Margin Trend | Multi-line | OPM% and net profit margin % |
| 8 | Health Score Gauge | Doughnut | ML score as a speedometer with colour zones |

Chart data is fetched via AJAX from `/api/v1/companies/{symbol}/charts/` with loading
states handled client-side. Redis caches the heaviest endpoints at a 60-minute TTL.

### Channel partner API

Authentication uses **HMAC-SHA256**. Partners send `X-API-Key-ID`, `X-Timestamp`,
`X-Signature` and `X-Nonce`; the server verifies with `secrets.compare_digest()`.

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/partner/v1/companies/{symbol}/full/` | Full financial dump for one company |
| GET | `/api/partner/v1/bulk-financials/` | `?symbols=TCS,INFY` — multi-company data |
| GET | `/api/partner/v1/screener/` | Screener with all filter params |
| GET | `/api/partner/v1/scores/` | ML health scores for all companies |
| POST | `/api/partner/v1/keys/` | Create an API key (secret shown once only) |
| POST | `/api/partner/v1/webhooks/` | Subscribe to `score_updated` / `anomaly_flagged` |

**Rate limits by tier**

| Tier | Per minute | Per hour | Per day |
|------|-----------|----------|---------|
| BASIC | 10 | 100 | 500 |
| PRO | 60 | 1,000 | 10,000 |
| ENTERPRISE | 300 | 10,000 | 1,00,000 |

---

## 9. Week-by-week plan

| Week | Focus | Key output |
|------|-------|-----------|
| **1** | Data engineering foundation | All ETL scripts, star schema live in PostgreSQL, all 100 companies loaded, quality checks passing |
| **2** | Exploratory data analysis | Notebook 1 with 20+ visualisations, correlation matrix, outlier analysis, written insight summary |
| **3** | Power BI part 1 | Dashboards 1 and 2 complete, DAX measures library built, numbers cross-checked against pgAdmin |
| **4** | Power BI part 2 | Dashboards 3–7 complete, consistency pass across all seven, peer review |
| **5** | Python ML engine | All 6 notebooks, scores/anomalies/peer mapping in PostgreSQL, Celery tasks scheduled |
| **6** | Django backend & pages | Site live locally, DRF endpoints tested, all 8 charts rendering |
| **7** | Partner API | HMAC auth, tiered throttling, usage logging, webhook delivery end-to-end |
| **8** | Testing, security, polish | 75%+ coverage, `bandit`/`safety` clean, N+1 queries fixed, final refresh, README, demo |

<details>
<summary>Day-level detail</summary>

**Week 1** — D1 read the SQL file and map every field into `data/schema_notes.md`;
install Python, PostgreSQL, venv. D2 write the extractor, verify row counts, commit.
D3 write the cleaner — year standardisation, analysis parsing, null handling, computed
columns; test on all 100 companies. D4 design the full star schema in `etl/schema.sql`
with PKs, FKs and indexes; verify in pgAdmin. D5 write the loader, run quality checks —
all must pass before Week 2.

**Week 2** — D1 connect Jupyter via SQLAlchemy, build the null heatmap, summary stats.
D2 revenue/profit distributions, top and bottom 10 by every major metric, sector
boxplots. D3 correlation matrix with a written interpretation. D4 outlier analysis
(IQR and Z-score) on labelled scatter plots. D5 finalise with five key insights and
present to mentor.

**Week 3** — D1 connect Power BI, set relationships, build the DAX measures table.
D2 Dashboard 1, all three pages, cross-check against SQL. D3 Page 2.1, tested with
TCS, HDFCBANK and WIPRO. D4 Pages 2.2 and 2.3. D5 Page 2.4, full review, commit PBIX.

**Week 4** — D1 Dashboard 3. D2 Dashboard 4 (ML scores stubbed as CSV until Week 5).
D3 Dashboard 5 — get the OPM% conditional formatting exactly right. D4 Dashboards 6
and 7. D5 consistency review across all seven, peer feedback, fix, save.

**Week 5** — D1 health scoring. D2 anomaly detection, refresh Dashboard 4 with real
scores. D3 K-Means clustering with PCA. D4 peer comparison and forecasting.
D5 Celery task wrappers, Beat schedule, verify all 6 tasks and DB writes.

**Week 6** — D1 Django project, apps, models, `inspectdb`. D2 serializers, ViewSets,
router, Swagger via drf-spectacular. D3 templates — `base.html` with Tailwind CDN,
home, company list. D4 company detail with all 8 charts via AJAX. D5 compare and
screener pages, Redis caching at 60-minute TTL.

**Week 7** — D1 `api_management` app and models (bcrypt-hashed secrets). D2
`HMACAPIKeyAuthentication` class, tested with correct and tampered signatures.
D3 `PartnerTierThrottle` on Redis counters. D4 all partner endpoints with async
usage logging. D5 webhook dispatch tested end-to-end.

**Week 8** — D1 pytest suite including HMAC valid/expired/tampered cases. D2
`bandit -r . -ll` and `safety check`; `black .` and `isort .`. D3 Django Debug Toolbar
for N+1 queries → `select_related` / `prefetch_related`; target <200 ms with warm cache.
D4 final Power BI refresh and publish, write README. D5 live demo.
</details>

---

## 10. Security guidelines

- **Secrets** — `python-decouple`, everything in `.env`, `.env` in `.gitignore`.
  Required: `SECRET_KEY`, `DATABASE_URL`, `REDIS_URL`, `DEBUG`, `ALLOWED_HOSTS`,
  `PBI_GATEWAY_TOKEN`.
- **API keys** — bcrypt-hashed, never plain text, secret shown once at creation,
  `secrets.compare_digest()` for all signature comparison.
- **Power BI** — never store PostgreSQL credentials in PBIX files; use a Gateway with
  environment variables; apply Row-Level Security on shared dashboards.
- **Input validation** — all API input via DRF serializers; CSV imports validated for
  MIME type, size (<5 MB) and column names before processing.
- **Rate limiting** — public API 100/hour anonymous, admin 1000/hour, partner per tier,
  JWT login endpoint 5/minute hard limit.

---

## 11. Deliverables & evaluation

### Weekly deliverables

| Week | Required output |
|------|-----------------|
| 1 | ETL scripts complete, star schema live, quality checks passing, 100 companies loaded |
| 2 | EDA notebook with 20+ visualisations, written insight summary, peer review done |
| 3 | Dashboards 1 and 2 complete with working slicers |
| 4 | All 7 dashboards complete, all PBIX files committed |
| 5 | All 6 notebooks complete; scores, anomalies and peer mapping in PostgreSQL; Celery running |
| 6 | Django site live locally, public endpoints tested, all 8 charts rendering |
| 7 | Partner API with HMAC auth, rate limiting enforced, webhooks delivering |
| 8 | Tests passing at 75%+, security issues fixed, demo delivered, README complete |

### Scoring criteria

| Criterion | Points | What is assessed |
|-----------|-------:|------------------|
| Power BI dashboard quality | 30 | Correct, readable, professional charts; working slicers; accurate DAX |
| Python analytics & notebooks | 20 | EDA depth, sensible ML scores, clean commented code |
| Data engineering & ETL | 15 | Correct star schema, clean pipeline run, passing quality checks |
| Django web app | 15 | Pages rendering, charts loading, endpoints responding |
| Partner API security | 10 | HMAC working, secrets hashed, rate limiting enforced |
| Code quality & testing | 10 | Clean code, meaningful tests, 75%+ coverage, no hardcoded secrets |

---

## 12. Quick reference

### Key commands

```bash
python etl/01_extract_from_mysql.py     # SQL dump → CSV
python etl/02_clean_and_transform.py    # clean + derive columns
python etl/03_load_to_warehouse.py      # load star schema into PostgreSQL
jupyter notebook                        # notebook work
python manage.py runserver              # Django dev server
celery -A bluestock worker -l info      # Celery worker
celery -A bluestock beat -l info        # Celery Beat
pytest --cov=. --cov-report=html        # tests with coverage
black . && isort . && flake8 .          # format and lint
docker-compose up --build               # full local stack
```

### Critical business rules

1. All financial data is in **INR Crores** — label every value accordingly.
2. **TTM** (Trailing Twelve Months) is always displayed separately from historical years
   and **never** included in growth-rate calculations.
3. Nulls are common in early years (2013–2015). In Power BI use
   `IF(ISBLANK(...), BLANK(), ...)`; in Python and Chart.js **skip** nulls rather than
   plotting zero.
4. Year format is inconsistent across tables — normalise to `'MMM YYYY'` in ETL and use
   `dim_year[sort_order]` for all time-series ordering.
5. `companies` uses a **string** symbol as primary key (e.g. `'TCS'`). All joins use
   this string, not an integer ID.
6. In Power BI, filter TTM rows out of trend charts — they distort CAGR. Use
   `dim_year[is_ttm] = FALSE` in measure filters.
7. For banks (HDFCBANK, AXISBANK, BANKBARODA, BAJFINANCE, BAJAJFINSV) the `borrowings`
   field includes **customer deposits**, which are not traditional debt. Note this on
   dashboards — bank D/E is not comparable to non-financial companies.

---

*Bluestock Fintech — Nifty 100 Financial Intelligence Platform · Intern Project Documentation · 2026*
ing India Village Geographical Data.md…]()

