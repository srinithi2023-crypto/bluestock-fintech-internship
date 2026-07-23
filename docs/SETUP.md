[India Village Geographical Data API Setup.md](https://github.com/user-attachments/files/30297801/India.Village.Geographical.Data.API.Setup.md)[Uploadin# Local Setup Guide

Setup instructions for reproducing each project locally.

---

## Prerequisites

| Tool | Version | Needed for |
|------|---------|-----------|
| Python | 3.11+ | All projects |
| PostgreSQL | 15 | B100 warehouse |
| Redis | 7+ | B100 caching/throttling, Capstone API |
| Power BI Desktop | Latest | B100 and MF dashboards (Windows only) |
| Docker + Compose | Latest | Optional — spins up Postgres, Redis, Django, Celery together |
| Git | Any recent | Everything |

---

## 1. Clone and create the environment

```bash
git clone https://github.com/<your-username>/bluestock-fintech-internship.git
cd bluestock-fintech-internship

python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

pip install --upgrade pip
pip install -r requirements.txt
```

---

## 2. Environment variables

Copy the example file and fill in real values. **Never commit `.env`.**

```bash
cp .env.example .env
```

```ini
# .env
SECRET_KEY=change-me
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1

DATABASE_URL=postgresql://postgres:password@localhost:5432/bluestock_dw
REDIS_URL=redis://localhost:6379/0

PBI_GATEWAY_TOKEN=
```

Values are read with `python-decouple`:

```python
from decouple import config
SECRET_KEY = config("SECRET_KEY")
DEBUG = config("DEBUG", cast=bool, default=False)
```

---

## 3. Project 1 — B100 Fundamental Analysis

### Create the database

```bash
createdb bluestock_dw
psql -d bluestock_dw -f etl/schema.sql
```

### Run the ETL, in order

```bash
python etl/01_extract_from_mysql.py     # SQL dump  → data/raw/*.csv
python etl/02_clean_and_transform.py    # raw       → data/clean/*.csv
python etl/03_load_to_warehouse.py      # clean     → PostgreSQL star schema
```

The loader is **idempotent** (`ON CONFLICT DO UPDATE`), so re-running is safe. Data
quality checks run at the end — do not proceed if any fail.

### Notebooks

```bash
jupyter notebook
```

Run `notebooks/01_eda.ipynb` through `06_trend_forecasting.ipynb` in order. Notebooks
2–6 write results back into PostgreSQL.

### Django app

```bash
python manage.py migrate
python manage.py inspectdb > companies/models_generated.py   # first time only
python manage.py createsuperuser
python manage.py runserver
```

In two more terminals:

```bash
celery -A bluestock worker -l info
celery -A bluestock beat -l info
```

Swagger docs are at `/api/schema/swagger-ui/`.

### Power BI

Get Data → PostgreSQL Database → server `localhost:5432`, database `bluestock_dw` →
import all `dim_*` and `fact_*` tables → set up the relationships listed in the
[project doc](01-b100-fundamental-analysis.md#connection-and-model).

Do **not** save the database password in the PBIX file.

### Docker shortcut

```bash
docker-compose up --build
```

Brings up PostgreSQL, Redis, Django and Celery in one command.

---

## 4. Project 2 — Capstone Village Geo-Data API

```bash
# Database: NeonDB PostgreSQL (or local Postgres for development)
python manage.py migrate
python manage.py import_geodata --source data/raw/india_villages.csv
python manage.py runserver
```

Redis must be running for the caching layer to behave as designed — without it, the
sub-100 ms target will not be met.

Deployment targets Vercel. Configure environment variables in the Vercel dashboard
rather than in a committed file.

---

## 5. Project 3 — Mutual Fund Analytics

```bash
cd bluestock_mf_capstone

python scripts/etl_pipeline.py          # raw → processed → SQLite
python scripts/compute_metrics.py       # CAGR, Sharpe, Beta, VaR → CSVs
```

Or run everything at once:

```bash
python run_pipeline.py
```

Then open the notebooks in order (`01_data_ingestion` → `05_advanced_analytics`) and
`dashboard/bluestock_mf.pbix` in Power BI Desktop.

Optional live NAV refresh:

```bash
python scripts/live_nav_fetch.py        # pulls from mfapi.in
```

---

## 6. Quality checks before committing

```bash
black . && isort . && flake8 .          # format and lint
pytest --cov=. --cov-report=html        # tests + coverage report
bandit -r . -ll                         # security scan (HIGH severity)
safety check                            # dependency vulnerabilities
```

Coverage target is **75% minimum**. All HIGH severity `bandit` findings must be
resolved before pushing.

---

## 7. Troubleshooting

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `psycopg2` install fails | Missing Postgres dev headers | Install `libpq-dev` (Linux) or use `psycopg2-binary` |
| ETL row counts don't match the dump | Escaped quotes in the regex parse | Check the `INSERT` parsing for `\'` handling |
| Power BI shows blanks for early years | Nulls in 2013–2015 data | Expected — use `IF(ISBLANK(...), BLANK(), ...)`, don't plot zero |
| CAGR figures look inflated | TTM rows included in the trend | Filter with `dim_year[is_ttm] = FALSE` |
| Bank D/E looks absurdly high | `borrowings` includes customer deposits | Expected — annotate, don't compare banks to non-financials |
| Django company page slow | N+1 queries | Add `select_related` / `prefetch_related`; check Debug Toolbar |
| MF NAV series has gaps | Weekends and market holidays | Reindex to a full date range, then `ffill()` |
g India Village Geographical Data API Setup.md…]()

