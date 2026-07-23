[Mutual Fund Analytics.md](https://github.com/user-attachments/files/30297728/Mutual.Fund.Analytics.md)
# Mutual Fund Analytics

> An end-to-end analytics platform for Indian mutual funds — NAV history, investor
> transactions and scheme performance — running from raw file ingestion through a SQLite
> star schema to risk metrics and an interactive Power BI dashboard.

| | |
|---|---|
| **Start date** | 01 Jun 2026 |
| **Status** | Active — 95% complete |
| **Priority** | Medium |
| **Team size** | 7 |
| **Tasks** | 8 (all done) |

---

## 1. Task timeline

| # | Task | Priority | Due | Status |
|---|------|----------|-----|--------|
| 1 | Project Setup + Data Ingestion (ETL) | Low | 03 Jun 2026 | Done |
| 2 | Data Cleaning + SQL Database Design | Medium | 06 Jun 2026 | Done |
| 3 | Exploratory Data Analysis (EDA) | Medium | 11 Jun 2026 | Done |
| 4 | Fund Performance Analytics | Medium | 13 Jun 2026 | Done |
| 5 | Dashboard Development (Power BI / Tableau) | Medium | 16 Jun 2026 | Done |
| 6 | Advanced Analytics + Risk Metrics | Medium | 18 Jun 2026 | Done |
| 7 | Final Report + Presentation + Deployment | Medium | 21 Jun 2026 | Done |
| 8 | Deliverables & Evaluation Rubric | Medium | 22 Jun 2026 | Done |

---

## 2. Folder structure

```
bluestock_mf_capstone/
├── data/
│   ├── raw/                        ← original downloaded files
│   ├── processed/                  ← cleaned, merged CSVs
│   └── db/                         ← bluestock_mf.db (SQLite)
├── notebooks/
│   ├── 01_data_ingestion.ipynb
│   ├── 02_data_cleaning.ipynb
│   ├── 03_eda_analysis.ipynb
│   ├── 04_performance_analytics.ipynb
│   └── 05_advanced_analytics.ipynb
├── scripts/
│   ├── etl_pipeline.py
│   ├── live_nav_fetch.py
│   ├── compute_metrics.py
│   └── recommender.py
├── sql/
│   ├── schema.sql
│   └── queries.sql
├── dashboard/
│   └── bluestock_mf.pbix
├── reports/
│   ├── Final_Report.pdf
│   └── Presentation.pptx
└── README.md
```

---

## 3. Data cleaning rules

### `nav_history.csv`
- Parse dates to `datetime`
- Sort by `amfi_code` + `date`
- **Reindex to a full date range, then `ffill()`** to cover holidays and weekends
- Remove duplicates
- Validate `NAV > 0`

### `investor_transactions.csv`
- Standardise `transaction_type` to `SIP` / `Lumpsum` / `Redemption`
- Validate `amount > 0`
- Fix inconsistent date formats
- Check `kyc_status` against the allowed enum

### `scheme_performance.csv`
- Validate all return values are numeric
- Flag anomalies
- Check `expense_ratio` falls within 0.1%–2.5%

---

## 4. Database design (SQLite star schema)

| Type | Table | Grain |
|------|-------|-------|
| Dimension | `dim_fund` | 1 row per scheme |
| Dimension | `dim_date` | 1 row per calendar date |
| Fact | `fact_nav` | 1 row per fund per date |
| Fact | `fact_transactions` | 1 row per investor transaction |
| Fact | `fact_performance` | 1 row per fund per performance window |
| Fact | `fact_aum` | 1 row per fund per period |

Primary and foreign keys are defined explicitly in `sql/schema.sql`. Loading uses
SQLAlchemy `create_engine` with `df.to_sql()`, and row counts are verified against the
source CSVs after every load.

### Ten analytical SQL queries (`sql/queries.sql`)

1. Top 5 funds by AUM
2. Average NAV per month
3. SIP year-over-year growth
4. Transactions grouped by state
5. Funds with expense ratio below 1%
6. Redemption-to-inflow ratio by fund category
7. Most volatile funds by NAV standard deviation
8. Investor count by KYC status
9. Best and worst performing fund per category per year
10. Month-on-month net flow across the whole book

A **data dictionary** in Markdown documents every column, its data type, business
definition and source reference.

---

## 5. Analytics layer

### Fund performance analytics
CAGR, absolute and annualised returns, rolling return windows, benchmark-relative
performance, expense-ratio-adjusted returns.

### Risk metrics
**Sharpe ratio**, **Beta** against a benchmark index, **Value at Risk (VaR)**, maximum
drawdown, volatility over multiple windows.

### Advanced analytics
Cohort analysis of investor behaviour by entry period, and a fund recommender scoring
schemes against a user's risk profile and horizon.

---

## 6. Dashboard

A 4-page interactive Power BI report (`dashboard/bluestock_mf.pbix`). **Every page
carries at least two interactive filters** — this is an explicit requirement, not a
nice-to-have.

Typical page split: fund overview, performance comparison, risk profile, and investor
flows.

---

## 7. Deliverables & evaluation rubric

| ID | Deliverable | Format | Weight | Criteria |
|----|-------------|--------|-------:|----------|
| D1 | ETL pipeline script | `.py` | 15% | Runs without manual steps, error handling, clean code |
| D2 | SQLite database | `.db` | 10% | Correct schema, all data loaded, queries run |
| D3 | EDA notebook | `.ipynb` | 15% | Depth of analysis, chart quality, insights documented |
| D4 | Performance metrics | `.ipynb` + CSVs | 15% | Mathematical accuracy, correct Sharpe/Beta/VaR formulas |
| D5 | Interactive dashboard | `.pbix` / `.twbx` | 20% | Design quality, all 4 pages, slicers functional |
| D6 | Advanced analytics | `.ipynb` | 10% | VaR correctness, cohort analysis quality, recommender logic |
| D7 | Final report + slides | `.pdf` + `.pptx` | 15% | Professional quality, complete sections, clear findings |

### Bonus challenges (+10 marks each)

| ID | Challenge |
|----|-----------|
| B1 | Schedule the ETL as a cron job auto-fetching NAV from `mfapi.in` every weekday at 8 PM |
| B2 | Build a Streamlit web app as an alternative to Power BI |
| B3 | Monte Carlo simulation projecting NAV growth over 5 years with uncertainty bands |
| B4 | Markowitz efficient frontier portfolio optimisation for 5 selected funds |
| B5 | Automated HTML email report generator sending weekly performance summaries |

---

## 8. Final deliverables

**Report** (`reports/Final_Report.pdf`, 15–20 pages) — executive summary, data sources,
ETL design, EDA findings, performance analysis, dashboard screenshots, limitations,
recommendations.

**Presentation** (`reports/Presentation.pptx`, 12 slides) — title, problem & objective,
data sources, architecture, EDA highlights (×2), performance metrics (×2), dashboard
screenshots (×2), key findings, thank you.

**Code hygiene** — all Python scripts carry docstrings, debug prints removed, and a
master `run_pipeline.py` executes the full flow.

**Release** —

```bash
git add .
git commit -m "Final: Complete Bluestock MF Capstone"
git tag v1.0
git push --tags
```

---

## 9. Common mistakes to avoid

These were called out explicitly in the brief and are worth keeping visible:

| ❌ Mistake | ✅ Correct approach |
|-----------|--------------------|
| Hard-coding file paths | Use `pathlib.Path` or `os.path.join` |
| Not handling weekends/holidays in NAV | Always `ffill()` after reindexing to a full date range |
| Using calendar days for CAGR | Annualise with `252 / n_trading_days` |
| Dashboard with no slicers | Every page needs at least two interactive filters |
| Confusing AUM lakh crore with scheme-level AUM crore | Include units in column names |
| Committing `.db` files to GitHub | Add `*.db` to `.gitignore`, share `schema.sql` instead |

---

## 10. Self-review checklist

- [ ] All 8 objectives met
- [ ] All 7 deliverables submitted
- [ ] Code runs end-to-end without errors
- [ ] Dashboard loads and all slicers function
- [ ] Report reads as a professional document
- [ ] `README.md` covers setup, ETL execution, dashboard access and dataset descriptions
- [ ] Repo tagged `v1.0`, no `.db` or raw data committed
- [ ] (Optional) Dashboard published to Power BI Service or Tableau Public, URL in README

---

*Bluestock Fintech — Mutual Fund Analytics Platform · Intern Documentation · 2026*

