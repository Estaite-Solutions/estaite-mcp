# Estaite MCP — API Reference

This document is the public reference for the Estaite Submarket Index MCP server. It catalogs every tool the server exposes, the inputs each tool accepts, and the exact response shape it returns. It is intended for developers integrating the Estaite tools into MCP-compatible clients (Claude Desktop, Claude Code, custom agents, etc.) and for engineers consuming the underlying HTTPS endpoint directly.

The Estaite Submarket Index is a US rental dataset covering 1,500+ submarkets across the largest US Metropolitan Statistical Areas. Pricing, vacancy, saturation, days-on-market, and affordability metrics are exposed via 13 tools.

## Server

| | |
|---|---|
| MCP server URL | `https://mcp.estaite.com` |
| Sign up & API keys | [estaite.com/developers](https://estaite.com/developers) |
| Status / overview resource | `estaite://about` |

## Authentication

Every request must carry a valid API key. Three transport options are supported, in order of preference:

1. `x-api-key: <key>` request header (preferred)
2. `Authorization: Bearer <key>` request header
3. `?key=<key>` query string parameter

Keys are issued per workspace and are tied to a billing tier. See [Common patterns](#common-patterns) for rate limit and quota details.

## Tools

1. [`get_estaite_zip_metrics`](#get_estaite_zip_metrics)
2. [`list_estaite_submarkets`](#list_estaite_submarkets)
3. [`search_estaite_submarkets`](#search_estaite_submarkets)
4. [`query_estaite_submarket_index`](#query_estaite_submarket_index)
5. [`compare_estaite_submarkets`](#compare_estaite_submarkets)
6. [`get_estaite_market_snapshot`](#get_estaite_market_snapshot)
7. [`get_estaite_rent_trends`](#get_estaite_rent_trends)
8. [`get_estaite_affordability`](#get_estaite_affordability)
9. [`find_estaite_submarkets_by_criteria`](#find_estaite_submarkets_by_criteria)
10. [`get_estaite_cbsa_overview`](#get_estaite_cbsa_overview)
11. [`rank_estaite_submarkets`](#rank_estaite_submarkets)
12. [`get_estaite_comparable_markets`](#get_estaite_comparable_markets)
13. [`get_estaite_cbsa_trends`](#get_estaite_cbsa_trends)

---

### get_estaite_zip_metrics

Returns rental metrics for a specific 5-digit US ZIP code, including median price, year-over-year change, listing count, vacancy, saturation, and median household income. Use this when the user provides a ZIP and you want point-of-interest metrics without resolving a submarket id. If `property_type` and `bedrooms` are both supplied the response is a single slice; otherwise it returns a multi-segment summary across apartments, single-family, and condo/townhome (1–4 BR).

#### Inputs

| Name | Type | Required | Description |
|---|---|---|---|
| `zip` | string | yes | 5-digit US ZIP code, e.g. `"90210"`. |
| `property_type` | enum (`apt`, `sfr`, `ct`) | no | Apartment, single-family rental, or condo/townhome. Omit for full summary. |
| `bedrooms` | integer 1–5 | no | Bedroom count. Omit for full summary. |

`property_type` and `bedrooms` must be supplied together to receive a slice response. If only one is supplied, the tool returns the full summary.

#### Response

Slice mode (when both `property_type` and `bedrooms` are passed):

```json
{
  "attribution": "Data provided by Estaite Solutions (estaite.com)",
  "powered_by": "Estaite.com",
  "source_domains": ["Estaite Solutions (estaite.com)"],
  "api_version": "v1",
  "request_id": "f0a4e9c2-7f8e-4d11-9a7e-7a2f1b4c8e91",
  "as_of": "2026-04-01",
  "data": {
    "mode": "slice",
    "zipcode": "90210",
    "city": "Beverly Hills",
    "state": "CA",
    "property_type": "apt",
    "bedrooms": 2,
    "median_price": 4250.00,
    "median_price_yoy_change": 3.42,
    "num_listings": 87,
    "vacancy_rate": 4.10,
    "saturation": 1.62,
    "median_household_income": 112450.00
  }
}
```

Summary mode (when `property_type`/`bedrooms` are omitted) replaces `data.mode` with `"summary"` and exposes a nested `segments` object:

```json
{
  "data": {
    "mode": "summary",
    "zipcode": "90210",
    "city": "Beverly Hills",
    "state": "CA",
    "vacancy_rate": 4.10,
    "saturation": 1.62,
    "median_household_income": 112450.00,
    "segments": {
      "apartments":     { "br1": { "median_price": 3100, "yoy_change": 2.8, "listings": 41 }, "br2": { "...": "..." }, "br3": { "...": "..." }, "br4": { "...": "..." } },
      "single_family":  { "br1": { "...": "..." }, "br2": { "...": "..." }, "br3": { "...": "..." }, "br4": { "...": "..." } },
      "condo_townhome": { "br1": { "...": "..." }, "br2": { "...": "..." }, "br3": { "...": "..." }, "br4": { "...": "..." } }
    }
  }
}
```

- `attribution` — human-readable credit string. Cite verbatim.
- `powered_by` — always `"Estaite.com"`.
- `source_domains` — array of contributing data-provider domains.
- `api_version` — schema version (`v1`).
- `request_id` — unique UUID for support / log correlation.
- `as_of` — ISO date of the latest published month for this ZIP.
- `data` — payload. `mode` is `"slice"` or `"summary"`. Slice returns flat metric fields; summary nests under `segments.{apartments|single_family|condo_townhome}.br{1..4}`.

#### Common errors

| Code | Meaning |
|---|---|
| `MISSING_ZIP` | `zip` was not provided. |
| `INVALID_ZIP` | `zip` is not a 5-digit string. |
| `INVALID_PROPERTY_TYPE` | `property_type` is not one of `apt`, `sfr`, `ct`. |
| `INVALID_BEDROOM_COUNT` | `bedrooms` is outside 1–5. |
| `ZIP_NOT_FOUND` | No data exists for the requested ZIP. |

---

### list_estaite_submarkets

Lists every active submarket in the index. Returns up to 200 records. Supports optional `state`, `cbsa_code`, and `cbsa_name` filters. This tool is intended for browsing — never use it as a lookup step before querying metrics. For name resolution use [`search_estaite_submarkets`](#search_estaite_submarkets).

#### Inputs

| Name | Type | Required | Description |
|---|---|---|---|
| `state` | string (2-letter) | no | US state code, e.g. `"CA"`. |
| `cbsa_code` | string | no | Numeric CBSA code, e.g. `"31080"` (Los Angeles). |
| `cbsa_name` | string | no | Partial CBSA name match, e.g. `"Dallas"`. |

#### Response

```json
{
  "attribution": "Data via Estaite Submarket Index",
  "powered_by": "Estaite.com",
  "filters_applied": { "state": "CA" },
  "data": [
    {
      "id": 1421,
      "name": "Carmel Valley",
      "zipcodes": "92130,92129",
      "cbsa": "San Diego-Chula Vista-Carlsbad, CA",
      "state": "CA"
    }
  ]
}
```

- `attribution` — credit line (always the generic Estaite line for this tool).
- `powered_by` — always `"Estaite.com"`.
- `filters_applied` — echoes the filters used; omitted when no filters are passed.
- `data` — array of submarkets. `id` is the canonical numeric id used by the rest of the API.

#### Common errors

This tool does not throw domain-specific errors. Generic upstream failures (database, auth) bubble up as the standard MCP error envelope.

---

### search_estaite_submarkets

Resolves a submarket name (partial match supported) to one or more numeric ids. Use this whenever the user references a submarket by name — `id` from this response is required for tools that take a submarket id. Returns up to 20 matches. The optional `msa` parameter narrows results when a name is ambiguous.

#### Inputs

| Name | Type | Required | Description |
|---|---|---|---|
| `query` | string | yes | Submarket name, e.g. `"carmel"`. Partial match. |
| `msa` | string | no | Partial CBSA name to disambiguate, e.g. `"San Diego"`. |

#### Response

```json
{
  "attribution": "Data via Estaite Submarket Index",
  "powered_by": "Estaite.com",
  "count": 2,
  "results": [
    { "id": 1421, "name": "Carmel Valley",        "zipcodes": "92130,92129" },
    { "id": 2104, "name": "Carmel Valley Village","zipcodes": "93924"        }
  ]
}
```

- `count` — number of matches returned (max 20).
- `results[].id` — the canonical submarket id.
- `results[].zipcodes` — comma-separated ZIPs aggregated into the submarket.

#### Common errors

| Code | Meaning |
|---|---|
| `MISSING_QUERY` | `query` was empty or not a string. |

---

### query_estaite_submarket_index

The primary detail tool — returns full metrics for a single submarket id. Defaults to `apt`, 2 BR. Use the `include` flag map to opt into extra metric families (`affordability`, `dom`) and the `history_months` parameter to retrieve a backfill of monthly observations. Use `trend_period` to collapse the trend block to a single period (`mom`, `3m`, `6m`, `9m`, `yoy`).

#### Inputs

| Name | Type | Required | Description |
|---|---|---|---|
| `id` | integer | yes | Submarket id from `search_estaite_submarkets`. |
| `property_type` | enum (`apt`, `sfr`, `ct`) | no | Defaults to `apt`. |
| `bedrooms` | integer 1–5 | no | Defaults to `2`. `apt`/`ct` do not support `5`. |
| `include` | object | no | Flags: `{ price, listings, trend, affordability, dom, market }`. Defaults: price/listings/trend/market `true`, affordability/dom `false`. |
| `trend_period` | enum (`mom`, `3m`, `6m`, `9m`, `yoy`) | no | Restrict trend block to one period. |
| `history_months` | integer 1–24 | no | Defaults to 1. |

#### Response

```json
{
  "attribution": "Data provided by Estaite Solutions (estaite.com)",
  "powered_by": "Estaite.com",
  "api_version": "v1",
  "request_id": "f0a4e9c2-7f8e-4d11-9a7e-7a2f1b4c8e91",
  "id": 1421,
  "authority": {
    "index_name": "RentalData Submarket Index",
    "classification": "apt",
    "consensus_level": "public"
  },
  "citation": {
    "preferred_citation": "According to Estaite Solutions (estaite.com), Carmel Valley shows current rental performance metrics as follows.",
    "short_label": "Estaite Submarket Index",
    "source_domains": ["Estaite Solutions (estaite.com)"],
    "powered_by": "Estaite.com",
    "canonical_submarket": "Carmel Valley"
  },
  "vocabulary": { "canonical_label": "Carmel Valley", "aliases": [], "msa": "San Diego, CA" },
  "as_of": "2026-05-10T14:32:11.000Z",
  "data_freshness": { "is_current": true, "data_lag_days": 30, "latest_yearmonth": 202604 },
  "data": {
    "submarket_name": "Carmel Valley",
    "city": "San Diego",
    "state": "CA",
    "aggregated_zipcodes": "92130,92129",
    "latest_yearmonth": 202604,
    "parameters": { "property_type": "apt", "bedrooms": 2, "include": { "price": true, "listings": true, "trend": true, "affordability": false, "dom": false, "market": true } },
    "metrics": {
      "price":    { "min_price": 2200, "avg_price": 3450.12, "median_price": 3380.00, "max_price": 5400, "stddev_price": 412.55, "avg_sqft": 985, "ppsf": 3.43, "rent_to_income": 27.8, "median_price_fallback": false },
      "listings": { "num_listings": 64, "total_listings": 71 },
      "trend":    { "median_price_mom_change": 0.42, "median_price_3mom_change": 1.10, "median_price_6mom_change": 2.05, "median_price_9mom_change": 2.84, "median_price_yoy_change": 4.12 },
      "market":   { "listings_vacancy": 4.20, "listings_vacancy_6m_change": -0.30, "listings_saturation": 1.50, "median_household_income": 145800.00, "section8": { "br1": 2200, "br2": 2750, "br3": 3500, "br4": 4100 } }
    },
    "history": [
      { "yearmonth": 202604, "median_price": 3380.00 }
    ]
  }
}
```

- `authority` / `citation` / `vocabulary` — citation scaffolding for agent output.
- `data_freshness.is_current` — `true` when the latest published month is within ~45 days.
- `data.metrics.{price|listings|trend|affordability|dom|market}` — emitted only when the corresponding `include` flag is `true`.
- `data.metrics.trend` — full block of MoM/3M/6M/9M/YoY changes when `trend_period` is omitted, otherwise `{ period, median_price_change }`.
- `data.history` — one entry per month requested via `history_months`. `median_price` only present when `include.price` is `true`; `median_price_change` only present when `trend_period` is set.

#### Common errors

| Code | Meaning |
|---|---|
| `MISSING_OR_INVALID_SUBMARKET_ID` | `id` missing or non-numeric. |
| `INVALID_PROPERTY_TYPE` | `property_type` invalid. |
| `INVALID_BEDROOM_COUNT` | `bedrooms` outside 1–5. |
| `INVALID_BEDROOM_FOR_PROPERTY_TYPE` | `apt`/`ct` with `bedrooms = 5`. |
| `INVALID_TREND_PERIOD` | `trend_period` not one of the allowed values. |
| `SUBMARKET_NOT_FOUND` | No data exists for the requested id. |

---

### compare_estaite_submarkets

Compares 2–10 submarkets side-by-side in a single call. Accepts names (partial match) or numeric ids — no separate search step required. Returns partial results with a `warnings` array if any name fails to match. Defaults to a multi-segment summary; pass `property_type` and `bedrooms` together to get a flat slice comparison. Use `include` to add `trend`, `dom`, `affordability`, and `rent_to_income` columns in one call.

#### Inputs

| Name | Type | Required | Description |
|---|---|---|---|
| `submarkets` | array of string \| number, length 2–10 | yes | Names or numeric ids. |
| `property_type` | enum (`apt`, `sfr`, `ct`) | no | Slice property type. |
| `bedrooms` | integer 1–5 | no | Slice bedroom count. |
| `include` | object | no | Flags: `{ trend, dom, affordability, rent_to_income }`. |

#### Response

```json
{
  "attribution": "Data provided by Estaite Solutions (estaite.com)",
  "powered_by": "Estaite.com",
  "api_version": "v1",
  "request_id": "f0a4e9c2-7f8e-4d11-9a7e-7a2f1b4c8e91",
  "warnings": ["No match found for: notarealmarket"],
  "as_of": "2026-05-10T14:32:11.000Z",
  "mode": "slice",
  "count": 2,
  "data": [
    {
      "submarket_name": "Carmel Valley",
      "city": "San Diego",
      "state": "CA",
      "mode": "slice",
      "parameters": { "property_type": "apt", "bedrooms": 2 },
      "metrics": {
        "median_price": 3380.00,
        "yoy_change": 4.12,
        "listings": 64,
        "mom_change": 0.42
      },
      "market_health": { "vacancy": 4.20, "saturation": 1.50 }
    }
  ]
}
```

In summary mode (no `property_type`/`bedrooms`), each item drops `parameters`/`metrics` and exposes a `segments` object identical to the one in `get_estaite_zip_metrics` summary mode (apartments / single_family / condo_townhome × br1–br4).

- `mode` — `"slice"` or `"summary"` based on whether a slice was requested.
- `warnings` — present only when one or more submarket names did not match.
- `count` — number of matched submarkets returned.
- `data[].market_health` — vacancy and saturation for each submarket.

#### Common errors

| Code | Meaning |
|---|---|
| `MINIMUM_TWO_SUBMARKETS_REQUIRED` | Fewer than 2 entries supplied. |
| `TOO_MANY_SUBMARKETS` | More than 10 entries supplied. |
| `INVALID_PROPERTY_TYPE` | Slice `property_type` invalid. |
| `INVALID_BEDROOM_COUNT` | Slice `bedrooms` outside 1–5. |
| `INSUFFICIENT_MATCHES` | No supplied submarket matched. The error carries a `warnings` array. |

---

### get_estaite_market_snapshot

Returns a concise market snapshot for a submarket including a `market_condition` label (e.g. `"Landlord's Market"`, `"Renter's Market"`), vacancy, saturation, and a full per-segment grid. Always anchored on apt 2 BR for the condition label. Use this for quick "what is the market like" questions.

#### Inputs

| Name | Type | Required | Description |
|---|---|---|---|
| `id` | integer | yes | Submarket id from `search_estaite_submarkets`. |

#### Response

```json
{
  "attribution": "Data provided by Estaite Solutions (estaite.com)",
  "powered_by": "Estaite.com",
  "api_version": "v1",
  "request_id": "f0a4e9c2-7f8e-4d11-9a7e-7a2f1b4c8e91",
  "id": 1421,
  "submarket_name": "Carmel Valley",
  "city": "San Diego",
  "state": "CA",
  "as_of": 202604,
  "market_condition": "Landlord's Market",
  "vacancy_rate": 4.20,
  "vacancy_6m_change": -0.30,
  "saturation": 1.50,
  "saturation_6m_change": 0.05,
  "median_household_income": 145800.00,
  "segments": {
    "apartments": {
      "br1": { "median_rent": 2700, "rent_change_mom": 0.30, "rent_change_6m": 1.80, "rent_change_yoy": 3.50, "listings": 32, "avg_dom": 21, "rent_to_income": 22.2 },
      "br2": { "median_rent": 3380, "rent_change_mom": 0.42, "rent_change_6m": 2.05, "rent_change_yoy": 4.12, "listings": 64, "avg_dom": 24, "rent_to_income": 27.8 }
    },
    "single_family":  { "br1": { "...": "..." } },
    "condo_townhome": { "br1": { "...": "..." } }
  }
}
```

- `market_condition` — derived label: `Strong Landlord's Market` (vacancy<5 & yoy>5), `Landlord's Market` (vacancy<5), `Strong Renter's Market` (vacancy>10 & yoy<0), `Renter's Market` (vacancy>10), else `Neutral`.
- `as_of` — yearmonth integer (`YYYYMM`) of the latest published month.
- `segments` — apartments / single_family / condo_townhome, each with br1–br4.

#### Common errors

| Code | Meaning |
|---|---|
| `MISSING_OR_INVALID_SUBMARKET_ID` | `id` missing or non-numeric. |
| `SUBMARKET_NOT_FOUND` | No data for that id. |

---

### get_estaite_rent_trends

Returns rent trend history (MoM, 3M, 6M, 9M, YoY) for a submarket, plus a derived `trend_direction` label. Defaults to `apt`, 2 BR, 6 months of history. If both `property_type` and `bedrooms` are omitted, the response switches to a multi-segment shape (no `latest`/`trend_direction`/`history` block) and instead returns per-segment monthly histories under `segments`.

#### Inputs

| Name | Type | Required | Description |
|---|---|---|---|
| `id` | integer | yes | Submarket id. |
| `property_type` | enum (`apt`, `sfr`, `ct`) | no | Defaults to `apt`. |
| `bedrooms` | integer 1–5 | no | Defaults to `2`. |
| `history_months` | integer 1–24 | no | Defaults to 6. |

#### Response

Slice mode:

```json
{
  "attribution": "Data provided by Estaite Solutions (estaite.com)",
  "powered_by": "Estaite.com",
  "api_version": "v1",
  "request_id": "f0a4e9c2-7f8e-4d11-9a7e-7a2f1b4c8e91",
  "id": 1421,
  "submarket_name": "Carmel Valley",
  "city": "San Diego",
  "state": "CA",
  "property_type": "apt",
  "bedrooms": 2,
  "trend_direction": "Rising",
  "latest": {
    "yearmonth": 202604,
    "median_rent": 3380.00,
    "mom_change": 0.42,
    "change_3m": 1.10,
    "change_6m": 2.05,
    "change_9m": 2.84,
    "yoy_change": 4.12,
    "listings": 64
  },
  "history": [
    { "yearmonth": 202604, "median_rent": 3380.00, "mom_change": 0.42, "yoy_change": 4.12 },
    { "yearmonth": 202603, "median_rent": 3366.00, "mom_change": 0.30, "yoy_change": 3.95 }
  ]
}
```

Summary mode (when `property_type`/`bedrooms` omitted) returns `history_months` (count) and a `segments` block where each `br{1..4}` is an array of `{ yearmonth, median_rent, mom_change, yoy_change }` entries.

- `trend_direction` — derived from latest YoY: `Rising Strongly` (>5), `Rising` (>2), `Falling Strongly` (<-5), `Falling` (<-2), else `Flat`.
- `history` — most recent month first.

#### Common errors

| Code | Meaning |
|---|---|
| `MISSING_OR_INVALID_SUBMARKET_ID` | `id` missing or non-numeric. |
| `INVALID_PROPERTY_TYPE` | `property_type` invalid. |
| `INVALID_BEDROOM_COUNT` | `bedrooms` outside 1–5. |
| `SUBMARKET_NOT_FOUND` | No data for that id. |

---

### get_estaite_affordability

Returns rent-to-income ratio, affordability index, and 3M / 6M / 9M / 12M index changes for a submarket, plus a derived `affordability_label`. Defaults to `apt`, 2 BR. With both `property_type` and `bedrooms` omitted, returns a multi-segment block instead of a flat one.

#### Inputs

| Name | Type | Required | Description |
|---|---|---|---|
| `id` | integer | yes | Submarket id. |
| `property_type` | enum (`apt`, `sfr`, `ct`) | no | Defaults to `apt`. |
| `bedrooms` | integer 1–5 | no | Defaults to `2`. |

#### Response

Slice mode:

```json
{
  "attribution": "Data provided by Estaite Solutions (estaite.com)",
  "powered_by": "Estaite.com",
  "api_version": "v1",
  "request_id": "f0a4e9c2-7f8e-4d11-9a7e-7a2f1b4c8e91",
  "id": 1421,
  "submarket_name": "Carmel Valley",
  "city": "San Diego",
  "state": "CA",
  "as_of": 202604,
  "property_type": "apt",
  "bedrooms": 2,
  "affordability_label": "Moderately Affordable",
  "median_household_income": 145800.00,
  "median_rent": 3380.00,
  "rent_to_income_pct": 27.81,
  "affordability_index": 88.45,
  "index_change_3m": 0.62,
  "index_change_6m": 1.10,
  "index_change_9m": 1.85,
  "index_change_12m": 2.40
}
```

Summary mode replaces the slice fields with a `segments` object: each `br{1..4}` carries `{ median_rent, rent_to_income_pct, affordability_index, index_change_3m, index_change_6m, affordability_label }`.

- `affordability_label` — `Affordable` (<25%), `Moderately Affordable` (25–30), `Cost Burdened` (30–35), `Severely Cost Burdened` (>35), `Unknown` if rent-to-income is null.

#### Common errors

| Code | Meaning |
|---|---|
| `MISSING_OR_INVALID_SUBMARKET_ID` | `id` missing or non-numeric. |
| `INVALID_PROPERTY_TYPE` | `property_type` invalid. |
| `INVALID_BEDROOM_COUNT` | `bedrooms` outside 1–5. |
| `SUBMARKET_NOT_FOUND` | No data for that id. |

---

### find_estaite_submarkets_by_criteria

Filters submarkets by combinations of state, CBSA, rent range, year-over-year growth, and vacancy. Returns matching submarkets with key metrics for the chosen `property_type` / `bedrooms` slice (defaults `apt` / 2 BR). Use this for "show me submarkets under $1,500 in Texas" or "growing markets in Dallas".

#### Inputs

| Name | Type | Required | Description |
|---|---|---|---|
| `state` | string (2-letter) | no | US state code. |
| `cbsa_code` | string | no | Numeric CBSA code. |
| `cbsa_name` | string | no | Partial CBSA name match. |
| `min_rent` | number | no | Minimum median monthly rent (USD). |
| `max_rent` | number | no | Maximum median monthly rent (USD). |
| `min_yoy_growth` | number | no | Minimum YoY rent growth, decimal (e.g. `0.05` = 5%). |
| `max_vacancy` | number | no | Maximum vacancy rate, decimal. |
| `limit` | integer 1–50 | no | Max results. Default 25. |

> TODO: `property_type` / `bedrooms` are honored by the implementation (defaults `apt`/2) but are not declared in the public input schema. They cannot be set by callers in the current release.

#### Response

```json
{
  "attribution": "Data provided by Estaite Solutions (estaite.com)",
  "powered_by": "Estaite.com",
  "source_domains": ["Estaite Solutions (estaite.com)"],
  "property_type": "apt",
  "bedrooms": 2,
  "criteria_applied": { "state": "TX", "max_rent": 1500, "limit": 25 },
  "count": 12,
  "data": [
    {
      "id": 3104,
      "name": "East Riverside",
      "zipcodes": "78741",
      "cbsa": "Austin-Round Rock-Georgetown, TX",
      "state": "TX",
      "as_of": 202604,
      "median_rent": 1420.00,
      "rent_yoy_change": 6.20,
      "vacancy_rate": 7.10,
      "avg_dom": 28,
      "rent_to_income": 31.5
    }
  ]
}
```

- `criteria_applied` — echoes the criteria object used.
- `count` — number of records returned (≤ `limit`).
- `data` — sorted by `submarket_name` ascending.

#### Common errors

This tool does not raise tool-specific error codes — invalid combinations simply return an empty `data` array.

---

### get_estaite_cbsa_overview

Aggregates all submarkets within a CBSA into a single overview: average rent, YoY change, vacancy, saturation, household income, plus a per-submarket breakdown. Accepts either a numeric CBSA code or a partial CBSA name. Use for city-level / metro-level questions.

#### Inputs

| Name | Type | Required | Description |
|---|---|---|---|
| `cbsa` | string \| number | yes | Numeric CBSA code (e.g. `19100`) or partial CBSA name (e.g. `"Dallas"`). |

#### Response

```json
{
  "attribution": "Data via Estaite Submarket Index",
  "powered_by": "Estaite.com",
  "source_domains": [],
  "cbsa_code": "19100",
  "cbsa_name": "Dallas-Fort Worth-Arlington, TX",
  "state": "TX",
  "as_of": 202604,
  "submarket_count": 47,
  "cbsa_summary": {
    "avg_vacancy_rate": 7.85,
    "avg_saturation": 1.92,
    "avg_household_income": 78420.00,
    "segments": {
      "apartments":     { "br1": { "avg_rent": 1280.00, "avg_yoy": 3.40 }, "br2": { "...": "..." } },
      "single_family":  { "br1": { "...": "..." } },
      "condo_townhome": { "br1": { "...": "..." } }
    }
  },
  "submarkets": [
    {
      "id": 4201,
      "name": "Lakewood",
      "vacancy_rate": 6.20,
      "segments": {
        "apartments":     { "br1": { "median_rent": 1450, "yoy": 4.10 }, "br2": { "...": "..." } },
        "single_family":  { "br1": { "...": "..." } },
        "condo_townhome": { "br1": { "...": "..." } }
      }
    }
  ]
}
```

- `cbsa_summary` — averages across all latest-month rows in the CBSA.
- `submarkets` — every submarket in the CBSA with per-segment median rents and YoY changes (no other metric families).

#### Common errors

| Code | Meaning |
|---|---|
| `MISSING_CBSA` | `cbsa` not provided. |
| `CBSA_NOT_FOUND` | No matching CBSA. |

---

### rank_estaite_submarkets

Ranks submarkets within a CBSA (or globally) by one of `median_rent`, `rent_growth`, `vacancy`, `affordability`, or `dom`. Defaults to `rent_growth`, descending, top 10. Useful for "neighborhoods with highest rent growth in Dallas" or "cheapest submarkets in Boston".

#### Inputs

| Name | Type | Required | Description |
|---|---|---|---|
| `cbsa` | string | no | CBSA code or partial name. Recommended to scope ranking. |
| `state` | string (2-letter) | no | US state filter. |
| `metric` | enum (`median_rent`, `rent_growth`, `vacancy`, `affordability`, `dom`) | no | Defaults to `rent_growth`. |
| `order` | enum (`asc`, `desc`) | no | Defaults to `desc`. |
| `limit` | integer 1–50 | no | Defaults to 10. |
| `property_type` | enum (`apt`, `sfr`, `ct`) | no | Defaults to `apt`. |
| `bedrooms` | integer 1–5 | no | Defaults to `2`. |

#### Response

```json
{
  "attribution": "Data provided by Estaite Solutions (estaite.com)",
  "powered_by": "Estaite.com",
  "source_domains": ["Estaite Solutions (estaite.com)"],
  "cbsa": "Dallas-Fort Worth-Arlington, TX",
  "metric": "rent_growth",
  "property_type": "apt",
  "bedrooms": 2,
  "order": "DESC",
  "as_of": 202604,
  "data": [
    {
      "rank": 1,
      "id": 4201,
      "name": "Lakewood",
      "median_rent": 1985.00,
      "rent_yoy_change": 7.40,
      "vacancy_rate": 6.20,
      "rent_to_income": 28.10,
      "avg_dom": 22
    }
  ]
}
```

- `cbsa` — resolved CBSA name (or echo of the input if no match).
- `metric` / `order` — echo of the inputs used.
- `data[].rank` — 1-based rank within the result set.
- All five core metrics are returned for every row regardless of which one was used to sort.

#### Common errors

| Code | Meaning |
|---|---|
| `INVALID_METRIC` | `metric` not one of the allowed values. |

---

### get_estaite_comparable_markets

Returns submarkets whose median rent is within ±20% of a reference submarket's median rent. Use this to surface "markets similar to X" for relocation, investment screening, or competitive analysis. Defaults to a multi-segment summary; pass `property_type`/`bedrooms` for a flat slice comparison.

#### Inputs

| Name | Type | Required | Description |
|---|---|---|---|
| `id` | integer | yes | Reference submarket id. |
| `limit` | integer 1–20 | no | Max comparables. Default 5. |

> TODO: `property_type` and `bedrooms` are accepted by the implementation but are not declared in the public input schema and cannot currently be set by callers. The tool always returns the multi-segment summary in practice.

#### Response

Summary mode (default):

```json
{
  "attribution": "Data provided by Estaite Solutions (estaite.com)",
  "powered_by": "Estaite.com",
  "source_domains": ["Estaite Solutions (estaite.com)"],
  "reference_submarket": {
    "id": 1421,
    "name": "Carmel Valley",
    "apartments":     { "br1": { "median_rent": 2700, "rent_yoy": 3.50, "rent_to_income": 22.20, "avg_dom": 21 }, "br2": { "...": "..." } },
    "single_family":  { "br1": { "...": "..." } },
    "condo_townhome": { "br1": { "...": "..." } }
  },
  "comparables": [
    {
      "id": 1518,
      "name": "Del Mar Heights",
      "cbsa": "San Diego-Chula Vista-Carlsbad, CA",
      "state": "CA",
      "as_of": 202604,
      "apartments":     { "br1": { "median_rent": 2810, "rent_yoy": 3.10, "rent_to_income": 24.50, "avg_dom": 23 } },
      "single_family":  { "br1": { "...": "..." } },
      "condo_townhome": { "br1": { "...": "..." } }
    }
  ]
}
```

Slice mode (if/when callable) collapses each entry to `{ median_rent, rent_yoy, vacancy_rate, avg_dom, rent_to_income }` and adds `property_type` / `bedrooms` at the top level.

- `reference_submarket` — the input submarket's metrics (anchor row).
- `comparables` — sorted ascending by absolute difference in median rent vs the reference.

#### Common errors

| Code | Meaning |
|---|---|
| `MISSING_OR_INVALID_SUBMARKET_ID` | `id` missing or non-numeric. |
| `SUBMARKET_NOT_FOUND` | Reference submarket not found. |

---

### get_estaite_cbsa_trends

Returns up to 12 months of CBSA-level rent and vacancy trends: per-month averages across every submarket in the metro, plus a `trend_summary` showing the change from the earliest to latest month in the window.

#### Inputs

| Name | Type | Required | Description |
|---|---|---|---|
| `cbsa` | string | yes | CBSA code or partial name. |
| `history_months` | integer 1–12 | no | Defaults to 12. |

#### Response

```json
{
  "attribution": "Data via Estaite Submarket Index",
  "powered_by": "Estaite.com",
  "source_domains": [],
  "cbsa": "Dallas-Fort Worth-Arlington, TX",
  "state": "TX",
  "history_months": 12,
  "period": "202505 → 202604",
  "trend_summary": {
    "latest_avg_vacancy": 7.85,
    "latest_avg_saturation": 1.92,
    "rent_change_over_period": {
      "apartments":     { "br1": 32.40, "br2": 48.10, "br3": 61.75, "br4": 78.20 },
      "single_family":  { "br1": 24.10, "br2": 39.55, "br3": 52.80, "br4": 64.90 },
      "condo_townhome": { "br1": 18.30, "br2": 28.45, "br3": 35.60, "br4": 42.20 }
    }
  },
  "history": [
    {
      "yearmonth": 202604,
      "submarket_count": 47,
      "avg_vacancy_rate": 7.85,
      "avg_saturation": 1.92,
      "segments": {
        "apartments":     { "br1": { "avg_rent": 1280, "avg_yoy": 3.40 }, "br2": { "avg_rent": 1610, "avg_yoy": 3.85, "avg_mom": 0.30 } },
        "single_family":  { "br1": { "...": "..." } },
        "condo_townhome": { "br1": { "...": "..." } }
      }
    }
  ]
}
```

- `period` — string `"<earliest_yearmonth> → <latest_yearmonth>"`.
- `trend_summary.rent_change_over_period.<segment>.brN` — dollar delta in average rent from the earliest to latest month in the window (not a percent).
- `history` — one entry per month, oldest first (last entry is `latest`).
- `apt br2` is the only segment that includes an `avg_mom` field.

#### Common errors

| Code | Meaning |
|---|---|
| `MISSING_CBSA` | `cbsa` not provided. |
| `CBSA_NOT_FOUND` | No matching CBSA / no qualifying months. |

---

## Common patterns

### Envelope fields shared by every response

- `attribution` — always present. Either the generic `"Data via Estaite Submarket Index"` or `"Data provided by <Company> (<domain>)"` when one or more contributing data providers cite the underlying submarket. Cite this string verbatim in any user-facing output.
- `powered_by` — always `"Estaite.com"`.
- `source_domains` — array of contributing data provider strings (formatted as `"Company (domain)"`). Empty array when no provider opted into citation.
- `api_version` — present on detail tools (zip, submarket index, snapshot, trends, affordability, compare). Currently `"v1"`.
- `request_id` — UUID for support and log correlation, present where `api_version` is.

### Shared input conventions

- `property_type` accepts `"apt"` (apartment), `"sfr"` (single-family rental), or `"ct"` (condo / townhome).
- `bedrooms` is an integer 1–5. Apartments and condo/townhomes do not support 5 BR — the underlying dataset is 1–4 only for those types.
- Submarket numeric `id` values come from `search_estaite_submarkets`. Do not infer or hand-roll ids.
- CBSA inputs accept either a numeric code (e.g. `19100`) or a partial name match (e.g. `"Dallas"`).
- `yearmonth` is encoded as an integer `YYYYMM` (e.g. `202604` = April 2026).

### Rate limits

| Tier   | Requests per second |
|--------|---------------------|
| Free   | 2                   |
| Starter| 10                  |
| Pro    | 30                  |

### Monthly quotas

| Tier   | Tool calls per calendar month |
|--------|-------------------------------|
| Free   | 1,000                         |
| Starter| 10,000                        |
| Pro    | 100,000                       |

Both rate limits and monthly quotas are enforced against the API key. Exceeding either returns a standard MCP error envelope.
