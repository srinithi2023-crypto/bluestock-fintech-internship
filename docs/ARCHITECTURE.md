[India Village Architecture.md](https://github.com/user-attachments/files/30297738/India.Village.Architecture.md)
# Architecture — Shared Patterns

All three internship projects follow the same analytical architecture. This page
documents the shared pattern once so the individual project pages can stay focused on
what makes each one different.

---

## The common pipeline

```
   Raw source                ETL                Warehouse           Consumption
 ┌─────────────┐      ┌──────────────┐      ┌────────────┐      ┌──────────────┐
 │ SQL dump    │      │ 01 extract   │      │            │      │ Power BI     │
 │ CSV files   │ ───▶ │ 02 clean     │ ───▶ │ Star       │ ───▶ │ Notebooks    │
 │ Public APIs │      │ 03 load      │      │ schema     │      │ Django / API │
 └─────────────┘      └──────────────┘      └────────────┘      └──────────────┘
                             │                     ▲
                             │                     │
                      quality gate ────────────────┘
                   (fail = stop, don't proceed)
```

| Stage | B100 | Capstone | Mutual Fund |
|-------|------|----------|-------------|
| Source | MariaDB SQL dump, 7 tables | Government geo-data files | NAV / transaction / performance files, `mfapi.in` |
| ETL | 3 Python scripts | Django management command | `etl_pipeline.py` |
| Warehouse | PostgreSQL 15 star schema | NeonDB PostgreSQL hierarchy | SQLite star schema |
| Consumption | 7 Power BI dashboards + Django + partner API | REST API + admin/B2B portals | Power BI dashboard + notebooks |

---

## Why a star schema

Both the B100 and Mutual Fund projects restructure raw operational tables into
dimensions and facts before anything is built on top. Three reasons:

1. **Power BI performs badly against normalised operational schemas.** The engine is
   built around a single fact table surrounded by dimensions; anything else produces
   ambiguous filter paths and slow visuals.
2. **Derived metrics get computed once, in ETL, not repeatedly in DAX.** `debt_to_equity`
   living in the fact table means every dashboard, notebook and API call sees the same
   number.
3. **A dimension table is where inconsistency goes to die.** `dim_year` is the single
   place that resolves `Mar-24` / `Mar 2024` / `TTM` — nothing downstream has to know
   the source was messy.

### Design rules used throughout

- Dimensions hold descriptive attributes; facts hold measures and foreign keys
- Every fact has an explicit, documented **grain** — one row per what?
- `sort_order` integers, never string sorting, for time-series ordering
- Boolean flags (`is_ttm`, `is_half_year`) on the date dimension so filtering is
  declarative rather than pattern-matching on labels

---

## Idempotency

Every load script can be re-run without corrupting the warehouse. In PostgreSQL this is
`ON CONFLICT DO UPDATE`; in SQLite it is a truncate-and-reload within a transaction.

This matters more than it sounds. During development the pipeline gets re-run dozens of
times, and a loader that only works on an empty database silently becomes a loader
nobody trusts.

---

## The data quality gate

After loading, and *before* any dashboard or notebook work begins, a set of checks runs:

- Row counts match the source
- Every fact foreign key resolves to a dimension row
- No negative values where they are impossible (NAV, sales, total assets)
- Null rates per column are within expected bounds
- Computed columns match a spot-check recalculation

**If a check fails, downstream work stops.** Building a dashboard on unvalidated data
just means discovering the problem later, after more work has been layered on top of it.

---

## Null handling philosophy

Nulls are common in early years of financial data (2013–2015 in the B100 dataset) and
around market holidays in NAV series. Two different situations, two different rules:

| Situation | Rule |
|-----------|------|
| Genuinely missing financial data | **Skip the point.** In Power BI, `IF(ISBLANK(...), BLANK(), ...)`; in Python and Chart.js, drop rather than plot zero. A zero is a claim; a blank is an admission. |
| Structurally absent data (weekend NAV) | **Forward-fill.** The value did not change because no trading occurred, so carrying the last value forward is the correct model. |

---

## Security patterns

| Concern | Approach |
|---------|----------|
| Secrets | `python-decouple` + `.env`, never committed. `.env.example` documents required keys. |
| Machine-to-machine auth | API key ID + HMAC-SHA256 signature over timestamp and nonce; verified with `secrets.compare_digest()` |
| Human auth | JWT via `djangorestframework-simplejwt` |
| Secret storage | bcrypt-hashed; plaintext shown exactly once at creation |
| Rate limiting | Redis counters, enforced per subscription tier at the throttle layer |
| Input validation | DRF serializers for all input; file uploads validated for MIME type, size and columns before parsing |
| BI credentials | Never inside PBIX files — gateway with environment variables, plus Row-Level Security on shared reports |

---

## Caching strategy

Reference data and financial history are both **read-heavy and slow-changing**, which
makes them ideal caching candidates.

- B100: 60-minute TTL on the heaviest financial API responses
- Capstone: aggressive caching of the geographic hierarchy, which is effectively static —
  this is what makes the sub-100 ms target achievable
- Cache invalidation is event-driven where scores change (`score_updated` webhook),
  time-based everywhere else

---

## Visual consistency

Seven Power BI files that each look different are seven files, not one product. Both BI
projects enforce a shared standard:

- A fixed colour semantic — **green is always good, red is always bad**, across every
  chart in every file
- One font family, three defined sizes
- Consistent title and footer format
- Every page carries at least two working slicers

The colour semantic is the one worth being strict about. If red means "high debt" on one
page and "selected item" on another, the dashboard is actively misleading.

