---
name: Resolve company names to Credit Benchmark CB_IDs
description: Map free-text company names to Credit Benchmark entity identifiers (CB_IDs) with ranked, scored candidates.
api: openapi/credit-benchmark-consensus-data-openapi.yml
operations: [getToken, matchExternalEntities]
---

# Resolve company names to Credit Benchmark CB_IDs

Use this before analytics/data calls when your source data has company names rather than `CB_ID` values.

## Steps

1. **Get a token** — `POST /api/security/token` (`getToken`); send `Authorization: Bearer <token>` afterward.
2. **Match names** — `POST /matching/text/match_external` (`matchExternalEntities`) with your list of external entities. For each input row you get back ranked `candidates`, each with a `CB_ID` and a confidence score.
3. **Select** — keep the highest-confidence `CB_ID` per input (apply your own confidence threshold), then reuse those CB_IDs as inputs to `getData` and the analytics routes.

## Rules
- Matching is entitlement-gated: `403` means the account cannot use resolution for that input; `422` means the request body failed validation.
- Confidence scores are relative rankings — always inspect them before trusting a single-candidate match.
