---
name: Query 30MHz sensor data
description: Find the data sources (checks) in a 30MHz organization and read their time-series measurements over a date range.
api: openapi/30mhz-zensie-openapi.json
operations: [getOrganizations_1, getOrganizationChecks, getCheck, getDataForSensors, getCheckStatsByTimeInterval, getChecksStatsByTimeInterval, getCheckLastRecordedState]
generated: '2026-09-05'
method: generated
source: openapi/30mhz-zensie-openapi.json + https://support.30mhz.com/query-data-from-the-30mhz-platform
---

# Query 30MHz sensor data

Read measurements out of the ZENSIE platform. Base URL `https://api.30mhz.com/api`.

## Before you start

Authentication is a bearer JWT API key. A human creates it once in the ZENSIE web app: profile menu →
Account Settings → Developer → **Request new API key**. There is no OAuth flow, no scopes and no
programmatic key issuance — you cannot mint this yourself. Send it on every call:

```
Authorization: Bearer <api-key>
```

An unauthenticated or malformed call returns `401 {"message":"Could not decode JWT token"}`.

## Step 1 — find your organization

`getOrganizations_1` (`GET /organization/user`) returns the organizations the key's user belongs to.
The `organizationId` also appears in the ZENSIE web URL, and it is required by almost every scoped
read below.

## Step 2 — find the checkIds

A **check** is a data source: one sensor channel, one imported feed, or one manual input sheet. Its
`checkId` is the primary key of the entire data surface.

`getOrganizationChecks` (`GET /check/organization/{organizationId}`) lists every check in the
organization with its sensor type and location. Use `getCheck` (`GET /check/{checkId}`) for one.

There is no pagination on this endpoint — it returns the full array. On a large multi-site grower that
response is big; cache it rather than re-fetching per read.

## Step 3 — read the data

Pick the operation that matches the shape you need:

- `getDataForSensors` (`POST /data/sensors`) — the general read. POST-bodied specifically so you can
  request many `checkIds` at once. Use this to pull a whole greenhouse in one call.
- `getCheckLastRecordedState` (`GET /stats/check/{checkId}`) — the latest value and state for one
  check. Use for a "what is it right now" question.
- `getCheckStatsByTimeInterval` (`GET /stats/check/{checkId}/from/{startDate}/until/{endDate}`) —
  aggregated statistics for one check over a range.
- `getChecksStatsByTimeInterval` (`POST /stats/from/{startDate}/until/{endDate}`) — the same for many
  checks.
- `getCheckStatsByInterval` (`GET /stats/check/{checkId}/interval/{interval}`) — a rolling interval
  instead of explicit dates. `interval` must be one of the API's allowed values; an invalid one
  returns `400 "must be one of valid allowed Intervals"`.
- `aggregate` (`POST /stats/aggregate`) — custom aggregation.

## Rules that will bite you

- **Dates are ISO 8601 with a Z offset.** Anything else returns
  `400 "The provided date must be a valid date: ISO 8601 date. E.g.: '2016-03-03T00:00:00Z'"`.
- **Bound your reads by date range, not by page.** Only two operations in the whole API accept
  `page`/`size`, and neither is a data endpoint. The date-range and interval path parameters are the
  windowing mechanism.
- **`timezoneOffset` is a real parameter** on 11 operations and changes how buckets are aligned. Set
  it deliberately; do not leave it to a default you have not read.
- **403 is the common failure, not 404.** 480 operations declare a 403. If a call fails, check the
  key's role in the organization before assuming the object is missing.
- **No rate limits are published.** There is no `RateLimit-*` header and no documented 429, so there
  is no runtime backoff signal. Be conservative and batch with `getDataForSensors` rather than
  looping per check.
- **Errors are `{"message": "<text>"}`**, not RFC 9457 problem+json, and the message text is not a
  stable code. Branch on the HTTP status, not on the string.

## Do not

- Do not use `getLocation`, `getPublicLocations`, `getAllPrivateLocations` or
  `getPrivateLocationsForOrganization` — the whole `location` vocabulary is deprecated in the
  contract. Use the `site` operations (`getSite`, `requestSite`) instead.
