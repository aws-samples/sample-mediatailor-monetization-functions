# Monetization Functions concepts

A short reference for the concepts every recipe in this library builds on. For
the authoritative documentation, see the
[MediaTailor User Guide](https://docs.aws.amazon.com/mediatailor/latest/ug/monetization-functions.html).

## What a function is

A **function** is a small, server-side unit of logic that MediaTailor runs at a
defined point during a playback session. Functions can:

- read session data (client IP, user agent, player parameters, SCTE-35 signals),
- call an external HTTP API,
- transform values with [JSONata](https://docs.aws.amazon.com/mediatailor/latest/ug/monetization-functions-jsonata.html)
  expressions, and
- write outputs that MediaTailor uses later in the ad-insertion flow.

You don't deploy or manage any infrastructure — the function definition lives on
your MediaTailor playback configuration.

## Lifecycle hooks

A function runs at a **lifecycle hook**. There are two:

| Hook | When it runs | Typical use |
| --- | --- | --- |
| `PRE_SESSION_INITIALIZATION` | Once, when a viewer starts a session | One-time setup: fetch an identity envelope, normalize the user agent |
| `PRE_ADS_REQUEST` | Before every ADS request (once per ad break) | Per-break targeting: enrich the request, split traffic, add headers |

A function is attached to a hook through a **function mapping** on the playback
configuration — a map of hook name to function ID.

## Function types

| Type | Purpose | External calls? |
| --- | --- | --- |
| `CUSTOM_OUTPUT` | Evaluate JSONata against session state and write outputs | No |
| `HTTP_REQUEST` | Make an outbound HTTP call, then evaluate outputs against the response | Yes |
| `SEQUENTIAL_EXECUTOR` | Run child functions in order, passing data forward between steps | Via children |
| `CONCURRENT_EXECUTOR` | Run child functions in parallel (up to the concurrency limit), then combine their outputs | Via children |

Executors compose other functions: a sequence or concurrent set can contain
functions, but those children cannot themselves be executors (maximum nesting
depth 2), with up to 10 child steps and 20 total function executions per
lifecycle hook. See the
[function types reference](https://docs.aws.amazon.com/mediatailor/latest/ug/monetization-functions-types.html)
for the full composition rules. Each recipe's `recipe.yaml` records the exact
`functionType` it uses.

## Output namespaces

Functions write to namespaced keys. The two you will use most:

- `player_params.*` — persisted to session state; available to later hooks and
  to your ADS URL via dynamic variable substitution.
- `temp.*` — passes data between functions within the same executor; not
  persisted after the hook completes.

## Recipe tiers: L1 vs L2

Recipes come in two shapes, recorded as `tier` in `recipe.yaml`:

- **L1** — the recipe is a direct function definition. `function.json` is the
  exact `PutFunction` body; copy, replace placeholders, deploy.
- **L2** — in the console, the recipe is authored through a higher-level form
  (for example, traffic-split rows) that compiles down to a function. In this
  library, an L2 recipe ships a **representative compiled** `function.json`
  produced from a canonical set of inputs, plus documentation of the parameters
  you would change. Edit the JSONata directly to customize.

## Fail-open behavior

If a function errors or times out, MediaTailor proceeds with the ad request
without the function's output — the viewer's stream is never interrupted. Design
your ADS URL and downstream logic to tolerate a missing output value.

## Deploying a recipe

Recipes are managed with two MediaTailor API operations (and their AWS CLI
equivalents):

- **`PutFunction`** — create or replace a function.
- **`PutPlaybackConfiguration`** — set the `FunctionMapping` that attaches the
  function to a playback configuration.

Each recipe README gives the exact steps for both the console and the CLI.

> **No built-in versioning.** Updating a function replaces its entire
> definition, and there is no server-side rollback. Keep the recipe files under
> source control (that is part of what this library is for) so you can restore a
> previous definition.
