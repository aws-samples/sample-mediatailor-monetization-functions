# User-Agent Normalization

Rewrite malformed or non-descriptive user-agents (SDK defaults like `okhttp`,
proxy-injected values, app-framework UAs) into well-formed strings, so ad
targeting and measurement work correctly.

- **Use case:** Data processing · Identity enrichment
- **Service:** Custom Output
- **Lifecycle hook:** `PRE_SESSION_INITIALIZATION`
- **Function type:** `CUSTOM_OUTPUT`
- **Tier:** L2 (see below)

## Scenario

Some players and SDKs send user-agents that ad decision servers can't interpret
(for example `okhttp/4.9.3`). At session start, this function checks the session
user-agent against a set of **case-sensitive substring rules** and, on a match,
rewrites it to a well-formed UA. It normalizes both `session.user_agent` and
`session.x_device_user_agent`.

## About this recipe (L2)

In the MediaTailor console this recipe is authored through a **rules form** —
each rule is *(match substring → replacement UA)* — which the console compiles
into the JSONata in `function.json`. This library ships the **compiled output**
for two illustrative rules so you can deploy it directly and edit the rules in
place. See [docs/concepts.md](../../docs/concepts.md#recipe-tiers-l1-vs-l2).

### Example rules shipped here

| If the UA contains… | …rewrite it to |
| --- | --- |
| `okhttp` | a well-formed Android Chrome UA |
| `CFNetwork` | a well-formed iOS Safari UA |

### How the expression works

Each output field is a right-associative chain of
`$contains(<field>, "<match>") ? "<rewrite>" : …`, ending in a fallback:

```text
{% $contains(session.user_agent, "okhttp") ? "<android-ua>"
   : $contains(session.user_agent, "CFNetwork") ? "<ios-ua>"
   : session.user_agent %}
```

- The fallback here is **keep-original** — if no rule matches, the field keeps
  its own value (so `session.x_device_user_agent` stays a no-op when the player
  doesn't send it). To use a fixed fallback instead, replace the trailing
  `session.user_agent` / `session.x_device_user_agent` with a quoted string.
- Rules must be **non-overlapping**: no match substring may contain another
  (for example `Fire` and `FireTV` would both match). The console blocks
  overlapping rules; keep that invariant when editing by hand.

To add a rule, insert another `$contains(<field>, "<match>") ? "<rewrite>" :`
term into both output fields.

## Values to replace

The shipped rules are illustrative. Edit `function.json` to match the malformed
user-agents your players actually send, and the well-formed strings you want to
substitute. There are no fixed placeholder tokens for this recipe.

## Deploy

### AWS CLI

```bash
aws mediatailor put-function --cli-input-json file://function.json

aws mediatailor put-playback-configuration \
  --name YOUR_PLAYBACK_CONFIG_NAME \
  --ad-decision-server-url "YOUR_ADS_URL" \
  --function-mapping PRE_SESSION_INITIALIZATION=user-agent-normalization
```

> `put-playback-configuration` replaces the whole configuration; include your
> existing settings so they are not cleared.

### Console

1. **Functions** → **Create function**, and enter the configuration from
   `function.json` (or use the console's User-Agent Normalization recipe form).
2. **Configurations** → your configuration → **Function mapping** → attach
   `user-agent-normalization` to the **Session initialization hook**.

## Test

See [test/](test/) for sample inbound user-agents and the normalized output.
