# Random Traffic Split Across Ad Servers

Randomly split ad requests across multiple ad decision server (ADS) URLs by
percentage — for A/B testing ad servers or spreading load across endpoints.

- **Use case:** Traffic management · Ad decisioning
- **Service:** Custom Output
- **Lifecycle hook:** `PRE_ADS_REQUEST`
- **Function type:** `CUSTOM_OUTPUT`
- **Tier:** L2 (see below)

## Scenario

Before each ad break, this function draws a random number and routes the ADS
request to one of several endpoints according to weights you choose. The example
here is an even 50/50 split between a primary and a secondary ADS URL: it
overrides `adsRequest.url` for that break.

## About this recipe (L2)

In the MediaTailor console this recipe is authored through a **traffic-split
form** — you add rows of *(ADS URL, weight %)* and the console compiles them into
the JSONata below. This library ships the **compiled output** for a canonical
50/50 two-way split so you can deploy it directly and edit the JSONata to add
rows or change weights. See [docs/concepts.md](../../docs/concepts.md#recipe-tiers-l1-vs-l2).

### How the expression works

```text
{% ($r := $random(); $r < 0.5 ? "<url1>" : $r < 1 ? "<url2>" : "<url2>") %}
```

- `$random()` returns a value in `[0, 1)`, bound once to `$r`.
- Thresholds are **cumulative weights ÷ 100**. With 50/50 the thresholds are
  `0.5` and `1.0`.
- Each row is a `$r < threshold ? url : …` test, right-associative. The trailing
  `else` repeats the last URL as a structural fallback.

To change the split, edit the thresholds and URLs. For a 70/30 split the first
threshold becomes `0.7`. To add a third endpoint, insert another
`$r < <cumulative> ? "<url>" :` term before the fallback. Keep the cumulative
weights summing to `1.0` so every draw maps to a URL.

## Values to replace

| Placeholder | Replace with |
| --- | --- |
| `https://ads-primary.example.com/vast` | Your first (primary) ADS URL |
| `https://ads-secondary.example.com/vast` | Your second (secondary) ADS URL |

## Deploy

### AWS CLI

```bash
aws mediatailor put-function --cli-input-json file://function.json

aws mediatailor put-playback-configuration \
  --name YOUR_PLAYBACK_CONFIG_NAME \
  --ad-decision-server-url "YOUR_DEFAULT_ADS_URL" \
  --function-mapping PRE_ADS_REQUEST=random-traffic-split
```

> This function overrides `adsRequest.url` per break. The configuration's own
> `--ad-decision-server-url` remains the default/fallback. `put-playback-configuration`
> replaces the whole configuration; include your existing settings.

### Console

1. **Functions** → **Create function**, and enter the configuration from
   `function.json` (or use the console's traffic-split recipe form directly).
2. **Configurations** → your configuration → **Function mapping** → attach
   `random-traffic-split` to the **Ad request hook**.

## Test

See [test/](test/) for how the output maps a random draw to an ADS URL.
