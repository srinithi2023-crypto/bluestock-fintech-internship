[Capstone Village API.md](https://github.com/user-attachments/files/30297708/Capstone.Village.API.md)

# Capstone Project — India Village-Level Geographical Data API

> A production-grade SaaS platform exposing India's complete village-level geographical
> data through a REST API, designed as backend infrastructure for B2B clients who need
> reliable, standardised address data for drop-downs and form autocomplete.

| | |
|---|---|
| **Start date** | 22 Apr 2026 |
| **Status** | Completed — 100% |
| **Priority** | High |
| **Team size** | 7 |
| **Tasks** | 5 (all done) |

---

## 1. Project goal

Build a SaaS platform providing a comprehensive REST API for India's complete
village-level geographical data. The platform serves as backend infrastructure for B2B
clients who otherwise have to build and maintain their own address databases.

---

## 2. Problem statement

Businesses building Indian e-commerce, logistics or service platforms face four
recurring problems:

- No standardised, reliable source for village-level address data
- Maintaining a local database requires constant updating and validation
- Drop-down menus with thousands of options cause front-end performance issues
- Address formats are inconsistent across regions

---

## 3. Solution

A centralised API platform that:

- Provides normalised hierarchical address data — **Country → State → District →
  Sub-District → Village**
- Returns a standardised format ready for drop-down menus:
  `Area Name (Village), Sub-District, District, State, India`
- Offers tiered access with usage-based pricing
- Includes admin controls and analytics for both the platform owner and B2B clients

---

## 4. Business value

| Stakeholder | Value delivered |
|-------------|-----------------|
| **B2B clients** | A ready-to-use API that removes the need to maintain a local geographical database |
| **End customers** | Consistent address format across every form and checkout |
| **Platform owner** | Recurring revenue through tiered subscriptions — Free, Premium, Pro, Unlimited |

---

## 5. Success criteria

- Import and normalise data for **all** Indian states, districts, sub-districts and villages
- **Sub-100 ms** API response time for 95% of requests
- Support **1M+ daily API requests** at full scale
- Intuitive dashboards for both administrators and B2B clients

---

## 6. Core features

| Category | Features |
|----------|----------|
| **Data coverage** | All Indian states, districts, sub-districts, villages |
| **API endpoints** | Search by name, filter by hierarchy, autocomplete |
| **Admin panel** | User management, API key issuance, analytics dashboards |
| **B2B portal** | Self-registration, API key generation, usage monitoring |
| **Security** | JWT authentication, API key + secret, rate limiting |
| **Scalability** | Redis caching, NeonDB PostgreSQL, Vercel edge deployment |

---

## 7. Architecture

```
                    ┌─────────────────────────┐
   B2B Client  ───▶ │  Vercel Edge (Django)   │
   (API key +       │  ─────────────────────  │
    secret)         │  JWT auth · rate limit  │
                    └───────────┬─────────────┘
                                │
                    ┌───────────▼─────────────┐
                    │      Redis cache        │  ← hot hierarchy lookups
                    └───────────┬─────────────┘
                                │ cache miss
                    ┌───────────▼─────────────┐
                    │  NeonDB (PostgreSQL)    │
                    │  states → districts →   │
                    │  sub_districts →        │
                    │  villages               │
                    └─────────────────────────┘
```

**Why each piece:**

- **NeonDB PostgreSQL** — serverless Postgres with branching, suits a read-heavy
  reference dataset
- **Redis** — the hierarchy is effectively static, so aggressive caching is what makes
  the sub-100 ms target realistic
- **Vercel edge deployment** — reduces round-trip latency for clients across India
- **API key + secret alongside JWT** — JWT for the human-facing portal, key/secret for
  machine-to-machine API traffic

---

## 8. Data model (hierarchy)

| Level | Notes |
|-------|-------|
| Country | Fixed — India |
| State | Includes union territories |
| District | Belongs to one state |
| Sub-District | Tehsil / taluk / block, belongs to one district |
| Village | Leaf node; the "area name" surfaced to end users |

Each level is a table with a foreign key to its parent, indexed on both the parent key
and the searchable name column. Autocomplete queries hit a name index scoped to the
selected parent, which keeps result sets small enough to return quickly.

---

## 9. Subscription tiers

| Tier | Intended user |
|------|---------------|
| Free | Evaluation and small projects |
| Premium | Growing platforms with steady traffic |
| Pro | Production e-commerce and logistics |
| Unlimited | High-volume enterprise integration |

Rate limits and quota enforcement are applied per tier at the API gateway layer, with
usage logged for the client-facing analytics dashboard.

---

## 10. Task breakdown

| Task | Priority | Due | Status |
|------|----------|-----|--------|
| Project Overview | Medium | 30 Apr 2026 | Done |
| Phase 1 | Medium | 30 Apr 2026 | Done |
| Phase 2 | Low | 30 Apr 2026 | Done |
| Skill Mastery & Capstone Project Execution | High | 29 Apr 2026 | Done |
| Frequently Asked Questions (FAQ) | Low | 30 May 2026 | Done |

---

## 11. Key takeaways

- **Reference data is a caching problem, not a query problem.** Once the hierarchy is
  normalised and indexed, almost all latency work happens in the cache layer.
- **Standardising output format is the actual product.** Clients could scrape this data
  themselves; what they are paying for is that every address comes back in the same shape.
- **Tiering has to be enforced at the edge**, not in application logic, or the rate
  limiter itself becomes the bottleneck at high request volumes.

---

*Bluestock Fintech — Capstone Project · Intern Documentation · 2026*
