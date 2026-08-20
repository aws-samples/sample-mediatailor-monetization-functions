# Gracenote OnConnect Lookup API by Title

Search [Gracenote](https://developer.tmsapi.com/) by program title and enrich ad
requests with the matching program's metadata. Use this when your player passes
a **content title** rather than a Gracenote TMS ID.

- **Use case:** Data processing
- **Service:** Gracenote · HTTP Request
- **Lifecycle hook:** `PRE_ADS_REQUEST` (recommended)
- **Function type:** `HTTP_REQUEST`

## Scenario

The player knows the **title** of the program being watched but not its
Gracenote TMS ID. Before each ADS request, this function searches the Gracenote
OnConnect Program Search API (`limit=1`), takes the first match, and writes its
metadata into `player_params.content_*` — the same content-metadata contract the
[TTD OpenAds video recipes](../) read.

If you already have the TMS ID, prefer
[gracenote-lookup-by-tms-id](../gracenote-lookup-by-tms-id/), which is an exact
lookup rather than a search.

## How it works

1. The player supplies `player_params.content_title`.
2. At the ad request hook, the function calls
   `https://data.tmsapi.com/v1.1/programs/search?q={title}&limit=1&api_key=...`.
3. On a `200` response with at least one result, it writes the first match's
   metadata to `player_params.content_*` (including the resolved
   `content_tms_id`, useful for caching or a follow-up details lookup). If there
   are no results, every output is `null`.

### Outputs

| Key | Source field | Notes |
| --- | --- | --- |
| `player_params.content_tms_id` | `[0].tmsId` | Resolved TMS ID of the match |
| `player_params.content_id` | `[0].tmsId` | Same value, per the content-metadata contract |
| `player_params.content_title` | `[0].title` | Canonical title (may differ from input) |
| `player_params.content_series` | `[0].seriesTitle` | Present on Episode entities |
| `player_params.content_season` | `[0].seasonNum` | Present on Episode entities |
| `player_params.content_episode` | `[0].episodeNum` | Present on Episode entities |
| `player_params.content_genre` | `[0].genres[0]` | Primary genre |
| `player_params.content_language` | `[0].titleLang` | e.g. `en`, `es` |
| `player_params.content_rating` | `[0].ratings[0].code` | e.g. `TV-14`, `PG-13` |
| `player_params.content_type` | `[0].entityType` | `Show`, `Episode`, or `Movie` |

## Prerequisites

- A Gracenote OnConnect account and API key from
  [developer.tmsapi.com](https://developer.tmsapi.com/).
- Your player must send `player_params.content_title`.

## Values to replace

| Placeholder | Replace with |
| --- | --- |
| `YOUR_GRACENOTE_API_KEY` | Your 24-character Gracenote OnConnect API key |

## Deploy

### AWS CLI

```bash
aws mediatailor put-function --cli-input-json file://function.json

aws mediatailor put-playback-configuration \
  --name YOUR_PLAYBACK_CONFIG_NAME \
  --ad-decision-server-url "YOUR_ADS_URL" \
  --function-mapping PRE_ADS_REQUEST=gracenote-lookup-by-title
```

> `put-playback-configuration` replaces the whole configuration; include your
> existing settings so they are not cleared.

### Console

1. **Functions** → **Create function**, and enter the configuration from
   `function.json`.
2. **Configurations** → your configuration → **Function mapping** → attach
   `gracenote-lookup-by-title` to the **Ad request hook**.

## Note on matching

Title search returns the best match, which may differ from the input in
capitalization or subtitle. The function surfaces the canonical title. For
deterministic results, resolve the title to a TMS ID once and switch to
[gracenote-lookup-by-tms-id](../gracenote-lookup-by-tms-id/).

## Security & privacy notes

This recipe places your Gracenote API key in the request URL and interpolates a
player-supplied value (`content_title`, passed through `$encodeUrlComponent`).
Before production, review
[docs/security-considerations.md](../../docs/security-considerations.md) — in
particular keeping your real API key out of source control, and treating
player-supplied input as untrusted.

## Test

See [test/](test/) for a sample input and the expected output.
