# LiveRamp ATS API

Enrich ad requests with a [LiveRamp](https://liveramp.com/) identity envelope
using the Authenticated Traffic Solution (ATS) API.

- **Use case:** Identity enrichment
- **Service:** LiveRamp · HTTP Request
- **Lifecycle hook:** `PRE_SESSION_INITIALIZATION`
- **Function type:** `HTTP_REQUEST`

## Scenario

At session start, the player passes a SHA-256 **hashed email** (plus the
viewer's consent signals). This function calls the LiveRamp ATS Envelope API to
resolve that hashed email into a privacy-safe **RampID envelope**, and stores it
in `player_params.liveRampEnvelope`. Because it runs once per session, every
subsequent ADS request can carry the envelope for cookieless, privacy-compliant
ad targeting — passed to your ad decision server via
[dynamic variable substitution](https://docs.aws.amazon.com/mediatailor/latest/ug/variables-domain.html).

## How it works

1. A viewer starts a playback session. The player supplies these session
   initialization parameters, surfaced to the function as `player_params.*`:
   - `hashed_email` — the viewer's SHA-256 hashed email address.
   - `consent_type` — LiveRamp consent type (`4` = TCF v2 for EU/EEA,
     `3` = CCPA for US legacy).
   - `consent_value` — the consent string from the player's CMP.
2. MediaTailor runs this function at the `PRE_SESSION_INITIALIZATION` hook and
   calls the ATS Envelope API. The request uses these query parameters:
   - `pid` — your LiveRamp placement ID.
   - `it=4` — identifier type `4` (hashed email).
   - `iv` — the URL-encoded hashed email.
   - `ct` / `cv` — the consent type and value.
   - `atype=3` — server-to-server implementation method.
3. On a `200` response, the function extracts the envelope value
   (`response.body.envelopes[type=19].value`) and writes it to
   `player_params.liveRampEnvelope`. On any other status it writes `null`.

## Prerequisites

- A LiveRamp account with the ATS Envelope API enabled, a **placement ID**, and
  an **approved origin domain**. See the
  [LiveRamp ATS API documentation](https://developers.liveramp.com/authenticatedtraffic-api).
- Your player must send `hashed_email`, `consent_type`, and `consent_value` as
  session initialization parameters.

## Values to replace

Replace these placeholders in [function.json](function.json) before deploying:

| Placeholder | Replace with |
| --- | --- |
| `YOUR_PLACEMENT_ID` | Your LiveRamp placement ID (from LiveRamp Console → Placements) |
| `YOUR_APPROVED_DOMAIN` | Your LiveRamp-approved origin domain (without the `https://` prefix) |

## Deploy

### AWS CLI

Create the function (replace placeholders in `function.json` first):

```bash
aws mediatailor put-function --cli-input-json file://function.json
```

Attach it to your playback configuration. `FunctionMapping` maps a lifecycle
hook to a function ID (see [mapping.json](mapping.json)). Merge this into a
`put-playback-configuration` call for your existing configuration:

```bash
aws mediatailor put-playback-configuration \
  --name YOUR_PLAYBACK_CONFIG_NAME \
  --ad-decision-server-url "YOUR_ADS_URL" \
  --function-mapping PRE_SESSION_INITIALIZATION=liveramp-ats-api
```

> `put-playback-configuration` replaces the whole configuration. Include your
> existing settings (ADS URL, and any other fields) so they are not cleared.

### Console

1. Open the MediaTailor console → **Functions** → **Create function**, and enter
   the configuration from `function.json`.
2. Go to **Configurations**, choose your playback configuration, and under
   **Function mapping** attach `liveramp-ats-api` to the **Session initialization
   hook**.

## Use the envelope in your ADS URL

Once the envelope is stored, reference it in your ad decision server URL:

```text
https://your-ads.example.com/vast?envelope=[player_params.liveRampEnvelope]
```

If the lookup failed, `player_params.liveRampEnvelope` is `null`; make sure your
ADS tolerates a missing or empty envelope value (functions
[fail open](../../docs/concepts.md#fail-open-behavior)).

## Security & privacy notes

This recipe forwards personal data (a hashed email, client IP, and consent
signals) to a third-party identity API. Before production, review
[docs/security-considerations.md](../../docs/security-considerations.md) — in
particular consent/PII obligations (GDPR/CCPA), keeping your real API token out
of source control, and that `session.client_ip` (forwarded as `X-Forwarded-For`)
can be spoofed upstream.

## Test

See [test/](test/) for a sample session-initialization request and the expected
function output.
