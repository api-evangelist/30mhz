---
name: Monitor a 30MHz sensor network
description: Check gateway connectivity, sensor voltage, uptime and alert state across a grower's 30MHz network.
api: openapi/30mhz-zensie-openapi.json
operations: [getNetworkStatusOverview, getStatusForGateways, getGatewaySensors, getGatewayClusterSensors, getVoltage, getCheckCondition, getCheckEventsForTimeRange, getNotificationsInOrganization, getCheckNotifications]
generated: '2026-09-05'
method: generated
source: openapi/30mhz-zensie-openapi.json
---

# Monitor a 30MHz sensor network

Answer "is this grower's network healthy, and if not, what broke".

## The sweep

1. `getNetworkStatusOverview` (`GET /stats/network/organization/{organizationId}`) — start here. One
   call, the whole organization's network state.
2. `getStatusForGateways` (`GET /gateway/status/{gatewayIds}`) — connectivity for a set of LoRa
   gateways. A gateway down takes every sensor behind it with it, so resolve gateway state before
   chasing individual sensors.
3. `getGatewaySensors` (`GET /check/gateway/{gatewayId}`) and `getGatewayClusterSensors`
   (`GET /check/gateway-cluster/{gatewayClusterId}`) — which checks sit behind a given gateway or
   cluster. This is how you translate "gateway X is down" into "these 40 data sources are stale".
4. `getVoltage` (`GET /sensor/{checkId}/voltage`) — battery voltage for a sensor. A declining
   voltage is the leading indicator of a sensor about to go silent.
5. `getCheckCondition` (`GET /stats/condition/{checkId}`) — the condition state of one check.
6. `getCheckLastRecordedState` (`GET /stats/check/{checkId}`) — a stale `timestamp` here is the
   simplest silent-sensor test.

## Alerts

- `getNotificationsInOrganization` (`GET /notification/organization/{organizationId}`) — the alert
  rules configured in the organization.
- `getCheckNotifications` (`GET /notification/check/{checkId}`) — the alerts on one check.
- `getCheckEventsForTimeRange` (`GET /stats/events/check/{checkId}/from/{startDate}/until/{endDate}`)
  and `getCheckEventsCountForTimeInterval`
  (`GET /stats/events/check/{checkId}/interval/{interval}`) — what actually fired.

## Rules that will bite you

- **Do not poll aggressively.** No rate limits are published and there is no 429 or `RateLimit-*`
  header to tell you when you are too close. Sweep on a schedule; do not hot-loop.
- **`getStatusForGateways` takes a LIST of gateway ids in the path.** Batch it rather than calling
  once per gateway.
- **There is no 30MHz status page.** `GET /status/ingest` is a service readiness probe (503 = "one or
  more backing services are unavailable") on the platform's own ingest path, not a public incident
  feed, and there is no status.30mhz.com. If the whole API is failing, you have no external signal to
  check — treat a broad 5xx as a platform incident and back off.
- **Alerts are in-platform notifications, not webhooks.** There is no event subscription. You are
  polling; design for that.
