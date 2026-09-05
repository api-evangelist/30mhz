---
name: Push external data into 30MHz
description: Create an import check and stream external measurements into the ZENSIE platform through the ingest surface.
api: openapi/30mhz-zensie-openapi.json
operations: [createImportCheck, getAllByOrganizationId, getIngestImportCheckTemplate, ingestImportChecks, updateManualImportChecks, deleteDataManualImportChecks, getImportCheck]
generated: '2026-09-05'
method: generated
source: openapi/30mhz-zensie-openapi.json + https://support.30mhz.com/push-data-to-the-30mhz-platform
---

# Push external data into 30MHz

Get measurements from a system 30MHz does not own — a climate computer, a third-party sensor, a lab
result — into ZENSIE so they sit alongside 30MHz's own sensors on the same dashboards.

## Step 1 — create an import check

An **import check** is a check whose data comes from outside. `createImportCheck`
(`POST /import-check/organization/{organizationId}`) creates one. The `ImportCheck` body carries the
schema of what you will send: `sensorType`, `unitPerField`, `decimalsPerField`, `rangePerJsonKey`,
`integrationType`, `timezone`, plus `locationId` / `zoneId` for where it lives.

Get the ones that already exist with `getAllByOrganizationId`
(`GET /import-check/organization/{organizationId}`) — do this first so you do not create a duplicate.

## Step 2 — get the call template

`getIngestImportCheckTemplate`
(`GET /ingest/organization/{organizationId}/import-check/{importCheckId}/example-call`) returns an
example ingest call for that import check. Read it before you build a payload — it is the closest
thing this API has to a dry run, and there is no validate-only mode.

## Step 3 — ingest

`ingestImportChecks` (`POST /ingest/organization/{organizationId}`) takes an array of
`ImportCheckEvent`: `checkId`, `timestamp`, `data`, optional `collectionId`, `unitsOverride`,
`updateData`.

**Hard limit: 100 events per call.** Exceeding it returns
`413 "Too many events sent. A maximum of 100 events are allowed per call"`. Chunk accordingly.

The response is `ImportCheckEventsResponse`, and it is a **partial-success** response:
`okEventsNo`, `failedEventsNo`, `failedEvents[]` (each an `ImportCheckEventFailure` with `errorCode`
and `errorText`), and `ingestedDocId`. A `200` does **not** mean every event landed — always read
`failedEventsNo` and re-drive only the failures.

## Correcting and deleting

- `updateManualImportChecks` (`POST /ingest/manual-input-update`) and
  `updateManualImportChecksWithId` (`POST /ingest/manual-input-update/{id}`) revise submitted rows.
- `deleteDataManualImportChecks` (`POST /ingest/manual-input-delete`) removes them.
- `deleteManualInputRowById` (`DELETE /import-check/{checkId}/manual-input-data/{rowId}`) removes one
  row.

## Rules that will bite you

- **There is no idempotency key.** Nothing in the 558-operation contract accepts an `Idempotency-Key`
  header, and the docs say nothing about retries. If an ingest call times out you cannot safely blind-
  retry it — read back with `getImportCheckCollectionValues` or `getManualInputDataForCheck` and
  reconcile before re-sending, or you will double-write measurements.
- **Timestamps are ISO 8601 with a Z offset.**
- **Deletes here are not reversible.** `deleteDataManualImportChecks` and `deleteManualInputRowById`
  have no restore operation and no recovery window is stated anywhere. Confirm with a human before
  deleting ingested data.
