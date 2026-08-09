---
name: Run Credit Benchmark portfolio analytics
description: Compute aggregate credit trends, breakdowns, rating changes, and rating distributions for a scoped universe of entities.
api: openapi/credit-benchmark-consensus-data-openapi.yml
operations: [getToken, metadata_available_dates_v2_metadata_available_dates_get, aggregateTrend, creditBreakdown, entityRatingChange, ratingDistribution]
---

# Run Credit Benchmark portfolio analytics

Computed analytics over a scoped universe. All routes relative to `https://gateway.creditbenchmark.com`; JWT bearer auth.

## Steps

1. **Get a token** — `POST /api/security/token` (`getToken`).
2. **Check effective dates** — `GET /analytics/v2/metadata/available-dates` (`metadata_available_dates_v2_metadata_available_dates_get`) for the latest and previous effective dates to anchor requests.
3. **Pick an analytic** (all take a `scope` = portfolio + filters):
   - **Aggregate trend** — `POST /analytics/v2/data/aggregatetrend` (`aggregateTrend`): time series of aggregate credit metrics over time.
   - **Credit breakdown** — `POST /analytics/v2/data/creditbreakdown` (`creditBreakdown`): credit-quality snapshot broken down by a facet column (sector, country, industry).
   - **Entity rating change** — `POST /analytics/v2/data/entityratingchange` (`entityRatingChange`): upgrades/downgrades over a lookback window.
   - **Rating distribution** — `POST /analytics/v2/data/ratingdistribution` (`ratingDistribution`): share of entities in each rating bucket over time.
4. **Interpret with metadata** — use `GET /analytics/v2/metadata/rating-scales` for CB21/CB7 band mappings, and the industry/geography schema routes for facet hierarchies.

## Rules
- A successful request can return an **empty result set** when `scope`/`result_filter` narrows the universe to nothing — that is not an error.
- `403` = not entitled to the requested universe/columns; `422` = validation error (inspect `detail[]`); `429` = rate limited, back off.
