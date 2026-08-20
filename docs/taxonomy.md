# Recipe taxonomy

Recipes are classified along the same two axes the MediaTailor console uses, so
that browsing here matches what you see in the console. Every recipe's
`recipe.yaml` carries both axes, which also makes the library machine-indexable.

## Axis 1 — Use case (`categories`)

A recipe belongs to one or more use-case categories:

| Category | Meaning |
| --- | --- |
| `IDENTITY_ENRICHMENT` | Resolve or enrich viewer identity (for example, cookieless identity envelopes) |
| `TRAFFIC_MANAGEMENT` | Route or split ad traffic across destinations |
| `DATA_PROCESSING` | Transform or normalize session data |
| `MONITORING_LOGGING` | Emit signals for observability and measurement |
| `AD_DECISIONING` | Influence which ads are requested or how bids are solicited |

## Axis 2 — Service (`services`)

The external or AWS services a recipe integrates with, plus the underlying
function primitive it uses (for example, `HTTP Request`, `Custom Output`).
Examples: `LiveRamp`, `Gracenote`, `The Trade Desk`.

## Matrix

Recipes plotted against use case (rows) and service (columns). A recipe appears
in every cell that matches one of its `categories` and one of its `services`.

| Use case \ Service | LiveRamp | Gracenote | The Trade Desk | Custom Output |
| --- | --- | --- | --- | --- |
| Identity enrichment | [LiveRamp ATS API](../recipes/liveramp-ats-api/) | | | [User-Agent Normalization](../recipes/user-agent-normalization/) |
| Traffic management | | | | [Random Traffic Split](../recipes/random-traffic-split/) |
| Data processing | | [Gracenote by TMS ID](../recipes/gracenote-lookup-by-tms-id/) · [by Title](../recipes/gracenote-lookup-by-title/) | | [User-Agent Normalization](../recipes/user-agent-normalization/) |
| Ad decisioning | | | [OpenAds: Gracenote US](../recipes/ttd-openads-video-gracenote-us/) · [Gracenote EU/UK](../recipes/ttd-openads-video-gracenote-eu/) · [Asset US](../recipes/ttd-openads-video-asset-us/) · [Asset EU/UK](../recipes/ttd-openads-video-asset-eu/) | [Random Traffic Split](../recipes/random-traffic-split/) |

## Recipe composition

Several recipes are designed to work together in a pipeline within a session:

1. **Identity** — [LiveRamp ATS API](../recipes/liveramp-ats-api/) at
   `PRE_SESSION_INITIALIZATION` writes `player_params.liveRampEnvelope`.
2. **Content metadata** — a Gracenote lookup
   ([by TMS ID](../recipes/gracenote-lookup-by-tms-id/) or
   [by Title](../recipes/gracenote-lookup-by-title/)) at `PRE_ADS_REQUEST` writes
   `player_params.content_*`.
3. **Bid solicitation** — a [TTD OpenAds recipe](../recipes/ttd-openads-video-gracenote-us/)
   at `PRE_ADS_REQUEST` reads both the identity envelope and content metadata.

The TTD "Asset Metadata" variants read `asset.*` from `EXT-X-ASSET` manifest
tags instead of a Gracenote lookup, so they skip step 2.

## Browsing by axis

- **By use case:** filter recipes whose `recipe.yaml` `categories` contains the
  category you want.
- **By service:** filter recipes whose `recipe.yaml` `services` contains the
  service you want.
