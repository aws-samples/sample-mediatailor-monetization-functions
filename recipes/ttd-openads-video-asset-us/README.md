# OpenAds: Video Ad via Ad Server, with Asset Metadata + LiveRamp (US)

Solicit bids for a video ad break from [The Trade Desk](https://www.thetradedesk.com/)
OpenAds, using server-side VAST caching. This variant targets **US (US Privacy / GPP)**
traffic and sources content metadata from EXT-X-ASSET manifest tags (`asset.*`).

- **Use case:** Ad decisioning
- **Service:** The Trade Desk · HTTP Request
- **Lifecycle hook:** `PRE_ADS_REQUEST`
- **Function type:** `HTTP_REQUEST`
- **Region / privacy:** US (US Privacy / GPP)
- **Content source:** EXT-X-ASSET manifest tags (`asset.*`)

## Scenario

Before each ad break, this function POSTs an OpenRTB 2.6 bid request to The Trade
Desk OpenAds auction endpoint. OpenAds caches the winning VAST server-side and
returns a **cache ID** and **cache URL**, which the function stores in
`player_params.ttd_cache_id` / `ttd_cache_url` (plus the bid price in
`ttd_bid_price`). You then pass the cache ID to your primary ad server for
server-side unwrapping — a header-bidding / OpenAds pre-bid pattern.

This recipe wires **LiveRamp** identity: if you also deploy the [LiveRamp ATS API recipe](../liveramp-ats-api/), it writes `player_params.liveRampEnvelope`, which this request surfaces as an OpenRTB `user.eids` entry (`source: liveramp.com`, `atype: 3`). Without it, `eids` is `null`.

## How it works

1. At `PRE_ADS_REQUEST`, the function builds an OpenRTB request from session
   state, SCTE-derived avail duration (`session.avail_duration_secs` drives
   `poddur`/`maxseq` for the pod), device signals, and content metadata.
2. It POSTs to `https://openads.adsrvr.org/openrtb2/auction` with a 550 ms
   timeout (slightly above the request's `tmax` of 500 ms).
3. On a `200` with a `seatbid`, it extracts:

   | Output | Source |
   | --- | --- |
   | `player_params.ttd_bid_price` | `seatbid[0].bid[0].price` |
   | `player_params.ttd_cache_id` | `seatbid[0].bid[0].ext.prebid.cache.bids.cacheId` |
   | `player_params.ttd_cache_url` | `seatbid[0].bid[0].ext.prebid.cache.bids.url` |

## Prerequisites

- A The Trade Desk OpenAds seller integration: a **publisher ID**, **supply
  source ID**, and an agreed **bid floor**.
- Your origin manifests must carry `EXT-X-ASSET` tags with the content metadata keys the request body reads (`CAID`, `TITLE`, `SERIES`, `SEASON`, `EPISODE`, `GENRE`, `CONTENTRATING`, `LANGUAGE`). Rename the `asset.*` reads in `function.json` to match your manifest's tag names.
- Recommended: the [LiveRamp ATS API recipe](../liveramp-ats-api/) deployed at
  session init for identity.

## Values to replace

| Placeholder | Replace with |
| --- | --- |
| `YOUR_TTD_PUBLISHER_ID` | Your TTD publisher ID |
| `YOUR_SUPPLY_SOURCE_ID` | Your TTD supply source ID |
| `YOUR_BIDFLOOR_NUMBER` | Your bid floor as a number (for example `15.0`) |

## Deploy

### AWS CLI

```bash
aws mediatailor put-function --cli-input-json file://function.json

aws mediatailor put-playback-configuration \
  --name YOUR_PLAYBACK_CONFIG_NAME \
  --ad-decision-server-url "YOUR_ADS_URL" \
  --function-mapping PRE_ADS_REQUEST=ttd-openads-video-asset-us
```

> `put-playback-configuration` replaces the whole configuration; include your
> existing settings so they are not cleared.

### Console

1. **Functions** → **Create function**, and enter the configuration from
   `function.json`.
2. **Configurations** → your configuration → **Function mapping** → attach
   `ttd-openads-video-asset-us` to the **Ad request hook**.

## Composing recipes

For a full pipeline, order the functions so metadata and identity are resolved
before the bid request:

1. [LiveRamp ATS API](../liveramp-ats-api/) at `PRE_SESSION_INITIALIZATION` (identity).
2. Ensure `EXT-X-ASSET` tags are present in your origin manifest (content metadata).
3. This recipe at `PRE_ADS_REQUEST` (bid solicitation) — chain via a
   `SEQUENTIAL_EXECUTOR` if you need it to run after the metadata step.

## Security & privacy notes

This recipe forwards device and identity data (client IP, `ifa`, user-agent,
LiveRamp envelope) and privacy signals to a third-party bidding API, and sources
content metadata from `EXT-X-ASSET` manifest tags (`asset.*`). Before
production, review
[docs/security-considerations.md](../../docs/security-considerations.md) — in
particular US consent obligations (the `us_privacy: "1---"` default is a
placeholder, not a consent decision), keeping your TTD credentials out of source
control, and validating/decoding `asset.*` values (their decoding by MediaTailor
is unverified for these samples).

## Test

See [test/](test/) for a sample input and the expected output.
