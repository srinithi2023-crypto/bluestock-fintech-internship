[India Village Data Dictionary.md](https://github.com/user-attachments/files/30297779/India.Village.Data.Dictionary.md)
# Data Dictionary

Field-level reference across all three projects.

**Units:** unless stated otherwise, all monetary values in the B100 project are in
**INR Crores**. Percentages are stored as numbers (11.0 means 11%), not decimals.

---

## Project 1 — B100 Fundamental Analysis

### `dim_company`

| Column | Type | Definition |
|--------|------|------------|
| `symbol` | VARCHAR(20) **PK** | NSE ticker, e.g. `TCS`. Used as the join key across every table. |
| `company_name` | TEXT | Full registered name, whitespace and `\r\n` stripped |
| `sector` | TEXT | Assigned sector — IT, Banking, NBFC, Insurance, Energy, Power, Ports, Cement, Healthcare, Auto, Paint, Consumer Goods, Holding Company |
| `sub_sector` | TEXT | Finer classification where available |
| `company_logo` | TEXT | Logo asset URL |
| `website` | TEXT | Corporate website |
| `nse_url` / `bse_url` | TEXT | Exchange listing pages |
| `face_value` | NUMERIC | Nominal share value in INR |
| `book_value` | NUMERIC | Net asset value per share in INR |
| `about_company` | TEXT | Descriptive summary |

### `dim_year`

| Column | Type | Definition |
|--------|------|------------|
| `year_id` | INT **PK** | Surrogate key |
| `year_label` | TEXT | Normalised display label, `'MMM YYYY'` format, e.g. `Mar 2024` |
| `fiscal_year` | INT | Financial year as an integer, e.g. `2024` |
| `quarter` | TEXT | `Q1`–`Q4` where applicable |
| `is_ttm` | BOOLEAN | True for Trailing Twelve Months rows. **Must be excluded from growth calculations.** |
| `is_half_year` | BOOLEAN | True for half-yearly rows |
| `sort_order` | INT | Correct chronological ordering. Always sort on this, never on `year_label`. |

### `dim_sector`

| Column | Type | Definition |
|--------|------|------------|
| `sector_id` | INT **PK** | Surrogate key |
| `sector_name` | TEXT | Display name, joins to `dim_company.sector` |
| `sector_code` | TEXT | Short code |
| `description` | TEXT | Sector notes |

### `dim_health_label`

| Column | Type | Definition |
|--------|------|------------|
| `label_id` | INT **PK** | Surrogate key |
| `label_name` | TEXT | `EXCELLENT` / `GOOD` / `AVERAGE` / `WEAK` / `POOR` |
| `min_score` / `max_score` | NUMERIC | Score band boundaries (0–100) |
| `color_hex` | TEXT | Display colour for consistent conditional formatting |

### `fact_profit_loss`
*Grain: one row per company per year*

| Column | Type | Definition |
|--------|------|------------|
| `symbol` | VARCHAR **FK** | → `dim_company` |
| `year_id` | INT **FK** | → `dim_year` |
| `sales` | NUMERIC | Total revenue, INR Cr |
| `expenses` | NUMERIC | Total operating expenses, INR Cr |
| `operating_profit` | NUMERIC | Sales − expenses, INR Cr |
| `opm_pct` | NUMERIC | Operating profit margin % |
| `other_income` | NUMERIC | Non-operating income, INR Cr |
| `interest` | NUMERIC | Interest expense, INR Cr |
| `depreciation` | NUMERIC | Depreciation and amortisation, INR Cr |
| `profit_before_tax` | NUMERIC | PBT, INR Cr |
| `tax_pct` | NUMERIC | Effective tax rate % |
| `net_profit` | NUMERIC | PAT, INR Cr |
| `eps` | NUMERIC | Earnings per share, INR |
| `dividend_payout_pct` | NUMERIC | Dividend as % of earnings |
| `net_profit_margin_pct` | NUMERIC | *Derived:* `(net_profit / sales) × 100` |
| `expense_ratio_pct` | NUMERIC | *Derived:* `(expenses / sales) × 100` |
| `interest_coverage` | NUMERIC | *Derived:* `operating_profit / interest`. Below 2.0 is a danger signal. |

### `fact_balance_sheet`
*Grain: one row per company per year*

| Column | Type | Definition |
|--------|------|------------|
| `equity_capital` | NUMERIC | Paid-up equity capital, INR Cr |
| `reserves` | NUMERIC | Reserves and surplus, INR Cr |
| `borrowings` | NUMERIC | Total borrowings, INR Cr. **For banks this includes customer deposits** and is not comparable to non-financial debt. |
| `other_liabilities` | NUMERIC | Remaining liabilities, INR Cr |
| `total_liabilities` | NUMERIC | Sum of liabilities, INR Cr |
| `fixed_assets` | NUMERIC | Net fixed assets, INR Cr |
| `cwip` | NUMERIC | Capital work in progress, INR Cr |
| `investments` | NUMERIC | Investment holdings, INR Cr |
| `other_assets` | NUMERIC | Remaining assets, INR Cr |
| `total_assets` | NUMERIC | Sum of assets, INR Cr |
| `debt_to_equity` | NUMERIC | *Derived:* `borrowings / (equity_capital + reserves)` |
| `equity_ratio` | NUMERIC | *Derived:* `(equity_capital + reserves) / total_assets`. Higher is stronger. |
| `book_value_per_share` | NUMERIC | *Derived:* `(equity_capital + reserves) / shares_outstanding` |

### `fact_cash_flow`
*Grain: one row per company per year*

| Column | Type | Definition |
|--------|------|------------|
| `operating_activity` | NUMERIC | Cash from operations, INR Cr |
| `investing_activity` | NUMERIC | Cash from investing, INR Cr. Usually negative for growing firms. |
| `financing_activity` | NUMERIC | Cash from financing, INR Cr. Negative = repaying debt. |
| `net_cash_flow` | NUMERIC | Sum of the three, INR Cr |
| `free_cash_flow` | NUMERIC | *Derived:* `operating_activity + investing_activity` |
| `cash_conversion_ratio` | NUMERIC | *Derived:* `operating_activity / net_profit`. Above 1.0 indicates high earnings quality. |

### `fact_analysis`
*Grain: one row per company per period*

| Column | Type | Definition |
|--------|------|------------|
| `period_label` | TEXT | `10Y` / `5Y` / `3Y` / `TTM`, parsed from source strings like `"10 Years: 11%"` |
| `compounded_sales_growth_pct` | NUMERIC | Revenue CAGR over the period |
| `compounded_profit_growth_pct` | NUMERIC | Profit CAGR over the period |
| `stock_price_cagr_pct` | NUMERIC | Share price CAGR over the period |
| `roe_pct` | NUMERIC | Average return on equity over the period |

### `fact_ml_scores`
*Grain: one row per company per scoring run*

| Column | Type | Definition |
|--------|------|------------|
| `computed_at` | TIMESTAMP | Run timestamp — enables score-trend-over-time charts |
| `overall_score` | NUMERIC | Composite health score, 0–100 |
| `profitability_score` | NUMERIC | Margin and return dimension |
| `growth_score` | NUMERIC | Revenue and profit growth dimension |
| `leverage_score` | NUMERIC | Debt and coverage dimension |
| `cashflow_score` | NUMERIC | Cash generation and conversion dimension |
| `dividend_score` | NUMERIC | Payout consistency dimension |
| `trend_score` | NUMERIC | Directional momentum dimension |
| `health_label` | TEXT | Bucketed label, joins to `dim_health_label` |

### `fact_pros_cons`
*Grain: one row per insight per company*

| Column | Type | Definition |
|--------|------|------------|
| `is_pro` | BOOLEAN | True = strength, False = concern |
| `category` | TEXT | Insight theme |
| `text` | TEXT | The insight itself |
| `source` | TEXT | `MANUAL` (pre-curated) or `ML` (generated) |
| `confidence` | NUMERIC | Model confidence for ML-sourced rows |
| `generated_at` | TIMESTAMP | Creation time |

---

## Project 2 — Capstone Village Geo-Data API

| Level | Key fields | Notes |
|-------|-----------|-------|
| State | `state_id`, `state_name`, `state_code` | Includes union territories |
| District | `district_id`, `district_name`, `state_id` FK | |
| Sub-District | `subdistrict_id`, `subdistrict_name`, `district_id` FK | Tehsil / taluk / block depending on region |
| Village | `village_id`, `village_name`, `subdistrict_id` FK | The "area name" shown to end users |

**Standardised output format:**
`Area Name (Village), Sub-District, District, State, India`

**Indexing:** each level is indexed on both its parent foreign key and its searchable
name column, so autocomplete queries stay scoped and fast.

---

## Project 3 — Mutual Fund Analytics

### `dim_fund`

| Column | Definition |
|--------|------------|
| `amfi_code` | AMFI scheme code — the join key across all fact tables |
| `scheme_name` | Full scheme name |
| `fund_house` | AMC name |
| `category` | Equity / Debt / Hybrid / Index etc. |
| `expense_ratio` | Annual expense ratio %. Validated to fall within 0.1%–2.5%. |
| `launch_date` | Scheme inception date |

### `dim_date`

| Column | Definition |
|--------|------------|
| `date` | Calendar date |
| `is_trading_day` | Distinguishes market days from weekends and holidays |
| `month`, `quarter`, `fiscal_year` | Derived time attributes for grouping |

### `fact_nav`
*Grain: one row per fund per date*

| Column | Definition |
|--------|------------|
| `amfi_code` FK | → `dim_fund` |
| `date` FK | → `dim_date` |
| `nav` | Net asset value. Validated `> 0`. Forward-filled across non-trading days after reindexing to a full date range. |

### `fact_transactions`
*Grain: one row per investor transaction*

| Column | Definition |
|--------|------------|
| `transaction_type` | Standardised to `SIP` / `Lumpsum` / `Redemption` |
| `amount` | Transaction value. Validated `> 0`. |
| `transaction_date` | Normalised date |
| `state` | Investor state — used for geographic flow analysis |
| `kyc_status` | Validated against the allowed enum |

### `fact_performance`
*Grain: one row per fund per performance window*

| Column | Definition |
|--------|------------|
| `return_1y`, `return_3y`, `return_5y` | Period returns %. All validated numeric; anomalies flagged. |
| `cagr` | Annualised using **252 / n_trading_days**, not calendar days |
| `sharpe_ratio` | Risk-adjusted return |
| `beta` | Sensitivity to the benchmark index |
| `var` | Value at Risk at the stated confidence level |
| `max_drawdown` | Largest peak-to-trough decline |

### `fact_aum`
*Grain: one row per fund per period*

| Column | Definition |
|--------|------------|
| `aum_crore` | Scheme-level assets under management, **in crore**. Note the unit explicitly — industry-level AUM is quoted in lakh crore and mixing the two is a common error. |

---

## Cross-project conventions

| Convention | Rule |
|------------|------|
| Currency | INR throughout. B100 in Crores; MF AUM in Crores at scheme level. |
| Percentages | Stored as numbers (`11.0` = 11%), never decimals |
| Dates | Normalised to `'MMM YYYY'` (B100) or ISO `datetime` (MF) during ETL |
| Ordering | Always via an integer `sort_order` or a real date column, never string sort |
| Nulls | Stored as true `NULL`/`NaN`, never the strings `'NULL'` or `'Null'` |
| Booleans | Real boolean columns (`is_ttm`, `is_trading_day`), not flag strings |
| Primary keys | B100 uses the string `symbol`; MF uses `amfi_code` |

