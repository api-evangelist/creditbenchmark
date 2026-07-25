---
name: Resolve entities and pull consensus credit analytics
description: Authenticate, resolve company names to Credit Benchmark CBIDs, then pull consensus rating distribution and a portfolio summary for the resolved universe.
api: openapi/creditbenchmark-openapi-original.yml
operations: [getToken, matchEntities, ratingDistribution, portfolioSummary]
---

# Resolve entities and pull consensus credit analytics

Use the Credit Benchmark REST API (`https://api.creditbenchmark.com`) to turn a list of
company names into consensus credit analytics.

## Auth
1. `getToken` — `POST /gartan/api/token` with `Username`, `Password`, `grant_type=password`
   (form-urlencoded). Read `access_token` from the JSON response; it is a JWT valid for 24h
   (`expires_in: 86400`).
2. Send `Authorization: Bearer <access_token>` on every subsequent call.

## Steps
1. `matchEntities` — `POST /matching/match` with your free-text company names. Take the top
   ranked candidate CBID per name (check the confidence score).
2. `ratingDistribution` — `POST /api/analytics/rating-distribution` scoped to those CBIDs to
   see the share of entities in each CB21/CB7 rating band.
3. `portfolioSummary` — `POST /api/analytics/portfolio-summary` for a consensus summary of the
   resolved portfolio.

## Rules
- Analytics routes wrap the typed request in a `RequestEnvelope`; set `scope`/`result_filter`,
  `metric`, and `rating_scale` (CB21 or CB7).
- A successful request can still return an **empty result set** when scope + result_filter
  narrow past coverage — that is expected, not an error.
- Errors use `{ "error": { "code", "message", "details" } }` (not RFC 9457); on `422` inspect
  `error.details[]`. `403` means the user is not entitled to the requested columns/data.
- No idempotency keys and no pagination — reads are safe to retry.
- Every response carries `metadata.request_id`; log it for support.
