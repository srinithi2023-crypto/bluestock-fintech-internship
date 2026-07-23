[India Village Geographical Data API README.md](https://github.com/user-attachments/files/30297640/India.Village.Geographical.Data.API.README.md)
# Bluestock Fintech — Data Analyst Internship

Documentation and deliverables from my Data Analyst internship at **Bluestock Fintech**
(Workspace `22A22J`), covering three end-to-end analytics projects built between
April and July 2026.

Each project follows the same lifecycle: **raw data → ETL → dimensional warehouse →
exploratory analysis → analytics/ML layer → BI dashboards → web/API delivery → report.**

---

## Projects

| # | Project | Domain | Core Stack | Status |
|---|---------|--------|-----------|--------|
| 1 | [B100 Intelligence — Fundamental Analysis](docs/01-b100-fundamental-analysis.md) | Equity research, Nifty 100 | Python · PostgreSQL · Power BI · Django REST | Active — 95% |
| 2 | [Capstone — India Village Geo-Data API](docs/02-capstone-village-api.md) | Geospatial SaaS platform | Django REST · NeonDB · Redis · Vercel | Completed — 100% |
| 3 | [Mutual Fund Analytics](docs/03-mutual-fund-analytics.md) | Fund performance & risk | Python · SQLite · Power BI | Active — 95% |

---

## What each project delivered

### 1. B100 Intelligence — Fundamental Analysis for Listed Companies
A financial intelligence system for India's top 100 listed companies, built from a
MariaDB export of ~12 years of company fundamentals.

- ETL pipeline parsing 7 raw tables into a **PostgreSQL star schema** (4 dimensions, 6 facts)
- **7 production Power BI dashboards** — Executive Overview, Company Deep Dive, Sector
  Comparison, Health Scorecard, Growth Analytics, Debt & Leverage, Dividend & Returns
- A **6-dimension ML health scoring model**, anomaly detection (Z-score + Isolation Forest),
  K-Means sector clustering, cosine-similarity peer mapping and ARIMA revenue forecasting
- A **Django + DRF web application** with 8 Chart.js visualisations per company, plus an
  HMAC-SHA256 authenticated channel-partner API with tiered rate limiting

→ [Full documentation](docs/01-b100-fundamental-analysis.md)

### 2. Capstone — India Village-Level Geographical Data API
A B2B SaaS platform exposing normalised, hierarchical Indian address data
(Country → State → District → Sub-District → Village) as a REST API.

- Import and normalisation of all Indian states, districts, sub-districts and villages
- Standardised address output for drop-down and autocomplete use cases
- JWT auth, API key + secret issuance, rate limiting, Redis caching
- Admin panel and self-service B2B portal with usage analytics
- Target: sub-100 ms response for 95% of requests, 1M+ daily requests at scale

→ [Full documentation](docs/02-capstone-village-api.md)

### 3. Mutual Fund Analytics
An analytics platform for Indian mutual fund NAV history, investor transactions and
scheme performance.

- ETL from raw NAV / transaction / performance files into a **SQLite star schema**
- Cleaning rules for holiday/weekend NAV gaps, transaction-type standardisation and
  expense-ratio validation
- Risk and performance metrics — **CAGR, Sharpe, Beta, VaR**, rolling returns, drawdown
- Cohort analysis, a fund recommender, and a 4-page interactive Power BI dashboard
- Final 15–20 page report and 12-slide presentation

→ [Full documentation](docs/03-mutual-fund-analytics.md)

---

## Repository layout

```
bluestock-fintech-internship/
├── README.md                          ← you are here
├── .gitignore
├── requirements.txt
└── docs/
    ├── 01-b100-fundamental-analysis.md
    ├── 02-capstone-village-api.md
    ├── 03-mutual-fund-analytics.md
    ├── ARCHITECTURE.md                ← shared patterns across all three projects
    ├── DATA_DICTIONARY.md             ← field-level reference
    ├── SETUP.md                       ← local environment setup
    └── EVALUATION_RUBRIC.md           ← deliverables and grading criteria
```

Project code lives in separate repositories (or sub-folders, depending on how you
clone this); the folder layout expected by each project is documented inside its
own page.

---

## Skills applied

**Data engineering** — SQL dump parsing, schema normalisation, star-schema design,
idempotent upserts, data quality gating, Celery-scheduled refreshes

**Analytics** — exploratory data analysis, correlation and outlier analysis, sector
benchmarking, financial ratio derivation, risk metrics (Sharpe, Beta, VaR, drawdown)

**Machine learning** — composite scoring models, Isolation Forest anomaly detection,
K-Means clustering with PCA visualisation, cosine-similarity peer matching, ARIMA forecasting

**Business intelligence** — Power BI data modelling, 25+ DAX measures, conditional
formatting systems, slicer-driven report design, unified visual standards

**Backend** — Django 4.2, Django REST Framework, JWT and HMAC-SHA256 authentication,
Redis caching and throttling, webhook dispatch, OpenAPI documentation

**Engineering practice** — Git workflow, pytest with 75%+ coverage targets, `bandit`
and `safety` security scanning, `black`/`isort`/`flake8`, Docker Compose

---

## Team

These were team projects delivered by a group of seven interns within the Bluestock
workspace. My contribution spanned the data engineering, EDA and Power BI dashboard
streams, alongside shared work on the analytics notebooks and final reporting. Task
assignments and progress were tracked in the Bluestock workspace.

---

## Notes

- All financial figures in the B100 project are in **INR Crores**.
- Data used in these projects was provided by Bluestock Fintech for training purposes.
  Raw datasets are **not** committed to this repository.
- Nothing here constitutes financial advice. These are educational analytics projects.

---

## Internship details

| | |
|---|---|
| **Organisation** | Bluestock Fintech |
| **Role** | Data Analyst Intern |
| **Workspace** | Bluestock 22A22J |
| **Duration** | April – July 2026 |
| **Projects** | 3 (16 tasks tracked) |

