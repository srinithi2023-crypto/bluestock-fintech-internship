[India Village Evaluation Rubric.md](https://github.com/user-attachments/files/30297850/India.Village.Evaluation.Rubric.md)
# Deliverables & Evaluation Rubric

Consolidated view of what each project was assessed on.

---

## Project 1 — B100 Fundamental Analysis

### Weekly deliverables

| Week | Required output |
|------|-----------------|
| 1 | All ETL scripts complete. Star schema in PostgreSQL. Data quality checks passing. All 100 companies loaded. |
| 2 | EDA notebook with 20+ visualisations. Written insight summary. Peer review done. |
| 3 | Dashboard 1 (Executive Overview) and Dashboard 2 (Company Deep Dive) complete, all pages with working slicers. |
| 4 | All 7 Power BI dashboards complete. All 7 PBIX files committed to Git. |
| 5 | All 6 Jupyter notebooks complete. Health scores, anomaly flags and peer mapping loaded to PostgreSQL. Celery tasks running. |
| 6 | Django website live locally. All public API endpoints tested. All 8 chart types rendering on the company detail page. |
| 7 | Channel partner API with HMAC auth. Rate limiting enforced. Webhook delivery working. |
| 8 | Tests passing at 75%+ coverage. All security issues fixed. Final demo delivered. README complete. |

### Scoring criteria — 100 points

| Criterion | Points | What is assessed |
|-----------|-------:|------------------|
| Power BI dashboard quality | **30** | Are charts correct, readable, professional? Do slicers work? Are DAX measures accurate? |
| Python analytics & notebooks | **20** | Is EDA thorough? Are ML scores sensible? Is code clean and commented? |
| Data engineering & ETL | **15** | Is the star schema correct? Does the pipeline run cleanly? Do quality checks pass? |
| Django web app | **15** | Are all pages rendering? Charts loading? Endpoints responding correctly? |
| Channel partner API security | **10** | HMAC working? Secrets properly hashed? Rate limiting enforced? |
| Code quality & testing | **10** | Clean code, meaningful tests, 75%+ coverage, no hardcoded secrets |

---

## Project 3 — Mutual Fund Analytics

### Deliverables — 100%

| ID | Deliverable | Format | Weight | Criteria |
|----|-------------|--------|-------:|----------|
| **D1** | ETL pipeline script | `.py` | 15% | Runs without manual steps, error handling, clean code |
| **D2** | SQLite database | `.db` | 10% | Correct schema, all data loaded, queries run |
| **D3** | EDA notebook | `.ipynb` | 15% | Depth of analysis, chart quality, insights documented |
| **D4** | Performance metrics | `.ipynb` + CSVs | 15% | Mathematical accuracy, correct Sharpe/Beta/VaR formulas |
| **D5** | Interactive dashboard | `.pbix` / `.twbx` | 20% | Design quality, all 4 pages, slicers functional |
| **D6** | Advanced analytics | `.ipynb` | 10% | VaR correctness, cohort analysis quality, recommender logic |
| **D7** | Final report + slides | `.pdf` + `.pptx` | 15% | Professional quality, complete sections, clear findings |

### Bonus challenges — +10 marks each

| ID | Challenge |
|----|-----------|
| **B1** | Schedule the ETL as a cron job auto-fetching NAV from `mfapi.in` every weekday at 8 PM |
| **B2** | Build a Streamlit web app as an alternative to Power BI |
| **B3** | Monte Carlo simulation projecting NAV growth over 5 years with uncertainty bands |
| **B4** | Markowitz efficient frontier portfolio optimisation for 5 selected funds |
| **B5** | Automated HTML email report generator sending weekly performance summaries |

---

## Project 2 — Capstone Village Geo-Data API

Assessed against the stated success criteria rather than a weighted rubric:

| Criterion | Target |
|-----------|--------|
| Data coverage | All Indian states, districts, sub-districts and villages imported and normalised |
| Latency | Sub-100 ms response for 95% of requests |
| Scale | Support 1M+ daily API requests |
| Usability | Intuitive dashboards for both administrators and B2B clients |

---

## Common failure modes across all projects

| ❌ Mistake | ✅ Correct approach |
|-----------|--------------------|
| Hard-coding file paths | `pathlib.Path` or `os.path.join` |
| Committing `.db` or raw data | `.gitignore` them; share `schema.sql` instead |
| Hardcoded credentials | `python-decouple` + `.env`, `.env` gitignored |
| Dashboards without slicers | Every page needs at least two interactive filters |
| Inconsistent colour meaning across pages | Green always good, red always bad — no exceptions |
| Including TTM rows in CAGR | Filter with `dim_year[is_ttm] = FALSE` |
| Using calendar days for annualisation | Use `252 / n_trading_days` |
| Plotting nulls as zero | Skip the point — a zero is a claim, a blank is an admission |
| Not handling weekends/holidays in NAV | Reindex to a full date range, then `ffill()` |
| Confusing lakh-crore and crore AUM | Put the unit in the column name |
| Comparing bank D/E to non-financials | Banks' `borrowings` includes customer deposits — annotate it |
