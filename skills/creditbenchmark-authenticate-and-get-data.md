---
name: Authenticate and extract Credit Benchmark data
description: Get a JWT token, resolve entity names to CB_IDs if needed, then extract raw consensus credit data for a scoped universe.
api: openapi/credit-benchmark-consensus-data-openapi.yml
operations: [getToken, matchExternalEntities, metadata_columns_v2_metadata_columns_get, getData]
---

# Authenticate and extract Credit Benchmark data

All routes are relative to `https://gateway.creditbenchmark.com`. Auth is a JWT bearer token; reuse it until it expires (do not re-authenticate on every call).

## Steps

1. **Get a token** — `POST /api/security/token` (`getToken`) with `Username` and `Password` as `application/x-www-form-urlencoded`. Keep the returned bearer token and send it as `Authorization: Bearer <token>` on every subsequent request.
2. **Resolve entities (only if you have names, not CB_IDs)** — `POST /matching/text/match_external` (`matchExternalEntities`) with your entity names. Take the top-ranked candidate's `CB_ID` for each name (check the confidence score).
3. **Discover available fields** — `GET /analytics/v2/metadata/columns` (`metadata_columns_v2_metadata_columns_get`) to see which columns you are entitled to and which are scopeable/facetable before building the request.
4. **Extract data** — `POST /analytics/v2/data/getdata` (`getData`) with an `effective_date` (or `lookback_period`/`lookback_unit`), a `scope` (portfolio + filters), the `columns` you want, and an optional `result_filter` (`limit`, `filters`, `sort`). The response is columnar arrays keyed by column name (`CB_ID`, `CB_Legal_Name`, `CB_CCR`, ...).

## Rules
- On `401`, the token is missing/expired — get a new one and retry.
- On `403`, the account is not entitled to a requested column or universe — reduce scope or contact support@creditbenchmark.com.
- On `422`, inspect the validation `detail[]` (loc/msg/type) to fix the offending field.
- On `429`, back off and retry; reuse the token instead of re-authenticating.
- Errors use a `{error, detail}` envelope (not RFC 9457).
