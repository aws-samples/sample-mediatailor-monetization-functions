# Gracenote OnConnect Lookup API by TMS ID

Resolve a [Gracenote](https://developer.tmsapi.com/) TMS ID to a full set of
program metadata for contextual ad targeting.

- **Use case:** Data processing
- **Service:** Gracenote · HTTP Request
- **Lifecycle hook:** `PRE_ADS_REQUEST` (recommended)
- **Function type:** `HTTP_REQUEST`

## Scenario

The player knows the Gracenote **TMS ID** of the program being watched (for
example `EP006883590001`) and passes it as a session parameter. Before each ADS
request, this function calls the Gracenote OnConnect Program Details API and
writes the returned metadata (title, series, season, episode, genre, language,
rating, entity type) into `player_params.content_*`.

Those keys are the exact content-metadata contract the
[TTD OpenAds video recipes](../) read, so placing this recipe ahead of an OpenAds
recipe auto-populates the OpenRTB `app.content.*` fields with no extra wiring.

## How it works

1. The player supplies `player_params.content_tms_id` — the TMS ID
   (for example `SH006883590000`, `EP006883590001`, `MV002523450000`).
2. At the ad request hook, the function calls
   `https://data.tmsapi.com/v1.1/programs/{tmsId}?api_key=...`.
3. On a `200` response it writes the metadata to `player_params.content_*`.
   Fields that don't apply to the program's entity type (for example
   `season`/`episode` for a movie) are written as `null`.

### Outputs

| Key | Source field | Notes |
| --- | --- | --- |
| `player_params.content_id` | `tmsId` | The TMS ID |
| `player_params.content_title` | `title` | Episode or program title |
| `player_params.content_series` | `seriesTitle` | Present on Episode entities |
| `player_params.content_season` | `seasonNum` | Present on Episode entities |
| `player_params.content_episode` | `episodeNum` | Present on Episode entities |
| `player_params.content_genre` | `genres[0]` | Primary genre |
| `player_params.content_language` | `titleLang` | e.g. `en`, `es` |
| `player_params.content_rating` | `ratings[0].code` | e.g. `TV-14`, `PG-13` |
| `player_params.content_type` | `entityType` | `Show`, `Episode`, or `Movie` |

## Prerequisites

- A Gracenote OnConnect account and API key from
  [developer.tmsapi.com](https://developer.tmsapi.com/).
- Your player must send `player_params.content_tms_id`.

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
  --function-mapping PRE_ADS_REQUEST=gracenote-lookup-by-tms-id
```

> `put-playback-configuration` replaces the whole configuration; include your
> existing settings so they are not cleared.

### Console

1. **Functions** → **Create function**, and enter the configuration from
   `function.json`.
2. **Configurations** → your configuration → **Function mapping** → attach
   `gracenote-lookup-by-tms-id` to the **Ad request hook**.

## Choosing a hook

This recipe is recommended at `PRE_ADS_REQUEST` so metadata is refreshed per ad
break (correct for live/linear where the program changes). If your content is
fixed for the whole session (for example a single VOD asset), you can attach it
at `PRE_SESSION_INITIALIZATION` instead to look it up once.

## Use the metadata in your ADS URL

```text
https://your-ads.example.com/vast?genre=[player_params.content_genre]&rating=[player_params.content_rating]
```

Values may be `null` if the lookup failed; ensure your ADS tolerates missing
values (functions [fail open](../../docs/concepts.md#fail-open-behavior)).

## Security & privacy notes

This recipe places your Gracenote API key in the request URL and interpolates a
player-supplied value (`content_tms_id`, passed through `$encodeUrlComponent`).
Before production, review
[docs/security-considerations.md](../../docs/security-considerations.md) — in
particular keeping your real API key out of source control, and treating
player-supplied input as untrusted.

## Test

See [test/](test/) for a sample input and the expected output.
