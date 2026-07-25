---
name: Track consensus rating moves across a universe
description: Authenticate, then identify entity-level upgrades and downgrades and a custom aggregate trend across a scoped entity universe.
api: openapi/creditbenchmark-openapi-original.yml
operations: [getToken, entityRatingChange, customAggregate]
---

# Track consensus rating moves across a universe

Surface where consensus credit quality is moving using the Credit Benchmark Analytics API
(`https://api.creditbenchmark.com`).

## Auth
1. `getToken` — `POST /gartan/api/token` (form-urlencoded `Username`/`Password`/`grant_type=password`).
2. Use `Authorization: Bearer <access_token>` (JWT, 24h) on all calls.

## Steps
1. `entityRatingChange` — `POST /api/analytics/entity-rating-change` with a scope and a lookback
   window to list entity-level upgrades and downgrades. Use `RatingChangeFilters` /
   `RatingChangeType` to filter to upgrades, downgrades, or both.
2. `customAggregate` — `POST /api/analytics/custom-aggregate` to compute an aggregate credit
   trend over the same scoped universe (choose the `metric` and `rating_scale`).

## Rules
- Wrap requests in the appropriate `RequestEnvelope_*`; set `scope`, `result_filter`, `metric`,
  `rating_scale`.
- Empty result sets are valid outputs of an over-narrow scope, not failures.
- Error envelope is `{ "error": { "code", "message", "details" } }`; handle `401` by refreshing
  the token, `403` as an entitlement gap, `422` by reading `error.details[]`.
- Contributor-data equivalents (`clientDataEntityRatingChange`, `clientDataCustomAggregate`)
  run the same analysis against your own submitted data.
