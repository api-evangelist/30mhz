---
name: Export 30MHz data in bulk
description: Create, track and retrieve a bulk data export from the ZENSIE platform, and register a recurring export partner push.
api: openapi/30mhz-zensie-openapi.json
operations: [getDataExportRowsCount, createDataExport, getDataExport, getDataExportForOrganization, deleteDataExport, createExportPartner, createExportPartnerJob, getExportPartnerJobsForOrganization]
generated: '2026-09-05'
method: generated
source: openapi/30mhz-zensie-openapi.json
---

# Export 30MHz data in bulk

For volumes past what `getDataForSensors` should carry, or for a recurring hand-off to another system.

## One-off export

1. `getDataExportRowsCount` (`POST /data-export/count`) — size the job **first**. It takes the same
   request shape as the export and returns `DataExportCountResponse.rowsCount`. Do this before
   creating an export you cannot estimate.
2. `createDataExport` (`POST /data-export`) — create it. The `DataExportRequest` body takes
   `checkIds`, `fromDate`/`toDate`, `aggregationInterval`, `aggregationMetric`, `documentType`,
   `format`, `decimalMark`, `timeZone`/`timezoneOffset`, `email`, and optional `location`,
   `sensorType`, `tags` filters.
3. `getDataExport` (`GET /data-export/{dataExportId}/organization/{organizationId}`) — poll it. The
   `DataExport` object carries `status`, `numberOfRows`, `duration` and `filename`; the response
   wrapper `DataExportResponse` carries `resourceUrl`, which is where the file actually lives.
4. `getDataExportForOrganization` (`GET /data-export/organization/{organizationId}`) — list past
   exports before creating another identical one.
5. `deleteDataExport` (`DELETE /data-export/{dataExportId}/organization/{organizationId}`) — clean up.

`decimalMark` matters: this is a European platform and a comma decimal mark will break a naive CSV
parser. Set it explicitly rather than accepting the default.

## Recurring push to another system

30MHz has no webhooks. The recurring mechanism is an **export partner**:

1. `createExportPartner` (`POST /export-partner/organization/{organizationId}`) — register the target
   with an `exportPartnerURL`.
2. `createExportPartnerJob` (`POST /export-partner-job`) — create the job: `exportPartnerId`,
   `exportPartnerURL`, `frequency`, `payload`, `active`. The platform pushes on that frequency and the
   job carries an `exportPartnerAccess` token (`token`, `tokenType`, `expiresAt`) for the receiver.
3. `getExportPartnerJobsForOrganization` (`GET /export-partner-job/organization/{organizationId}`) —
   list what is already running.
4. `removeExportPartnerJob` (`DELETE /export-partner-job/{jobId}/organization/{organizationId}`) —
   stop one.

**Understand what this is not.** It is a scheduled push, not an event subscription. There is no event
type, no delivery receipt, no retry policy and no payload signature documented anywhere in the
contract or the docs. Build the receiving end to tolerate duplicates and gaps, and verify the
`exportPartnerAccess` token yourself.

## Rules that will bite you

- Export creation is a write with no idempotency key. A retried `createDataExport` creates a second
  export and a second email. Check `getDataExportForOrganization` before re-creating.
- `deleteDataExport` and `removeExportPartnerJob` have no reversal operation and no stated recovery
  window. Confirm with a human.
