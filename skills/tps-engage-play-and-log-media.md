---
name: tps-engage-play-and-log-media
description: >-
  Drive a Blindspot (TPS Engage) DOOH player device: prefetch upcoming
  creatives, pull the creative to play now, and report a verified play via the
  popUrl proof-of-play callback.
api: Blindspot Pull API
generated: '2026-07-21'
method: generated
source: openapi/tps-engage-blindspot-pull-api-openapi.yaml
operations:
- GET /prefetch/{deviceId}
- GET /play/{deviceId}
---

# Play and log media on a Blindspot screen

The Blindspot Pull API is the device-facing surface of the TPS Engage DOOH
network: screens pull what to play, then log the play. Base URL:
`https://rtb.network.tpsengage.com/api/{apiPublisher}` (default publisher
segment `sv`). The published spec declares no authentication scheme — devices
are addressed by their registered `deviceId` (UUID). The spec defines no
operationIds; operations are identified here by method + path.

## Steps

1. **Prefetch upcoming creatives** — `GET /prefetch/{deviceId}`.
   Cache every `creativeUrl` in the returned array locally. An empty array is
   valid (nothing to preload). A `404` means the `deviceId` is not registered.

2. **Pull the creative to play now** — `GET /play/{deviceId}`.
   - `200` returns `bidId`, `creativeId`, `creativeUrl`, and `popUrl`.
   - `204` means nothing is scheduled: show fallback/house content and poll
     again for the next slot.

3. **Render the creative** from `creativeUrl` (transcoded MP4 hosted on
   Google Cloud Storage), preferring the locally prefetched copy.

4. **Report the verified play** — request the returned `popUrl` exactly as
   given. It is a prebuilt playlog callback that encodes the device
   (`location`), creative (`media`), `campaign`, `network`, and a bid payload
   (`bidId`, `audience`, `cpm`). Skipping this step means the play is never
   counted or billed.

## Error handling

- `404` on prefetch: verify the device is registered with Blindspot before
  retrying.
- Unknown/malformed deviceIds on `/play` have been observed returning `500`
  (probed 2026-07-21); treat as retryable with backoff, and fall back to
  house content meanwhile.
- Both operations are idempotent GETs — safe to retry.
