# Security & privacy considerations

These recipes are **learning examples that you own and customize** — not
turnkey, production-ready functions. They are intentionally simple so the
mechanics are easy to follow. Before you deploy a recipe, review it against your
own security posture, jurisdiction, and architecture, and change what your
context requires.

This document explains the decisions the samples deliberately leave to you, and
what to weigh for each. It is a checklist of *considerations*, not a set of
mandates — you are the owner of the deployed function.

> **Rule of thumb:** treat every recipe as a starting point. Read the JSONata,
> understand every value it reads and every place it sends data, and adapt it
> before it touches production traffic.

## Before you deploy any recipe

- **Read the whole function.** Know exactly which inputs it reads
  (`player_params.*`, `session.*`, `asset.*`, `scte.*`) and every external
  endpoint it calls.
- **Treat all player/session input as untrusted.** Values like
  `player_params.*`, `session.user_agent`, `session.client_ip`, and `asset.*`
  can be influenced by the client or upstream systems. Validate, encode, or
  constrain them as appropriate for how you use them.
- **Confirm the destinations.** Make sure every URL the function contacts is one
  you intend and control the relationship with.
- **Test with hostile input**, not just the happy-path sample values in each
  recipe's `test/` fixtures.

## 1. Handling secrets and credentials

Recipes use obvious placeholders (for example `YOUR_GRACENOTE_API_KEY`,
`YOUR_TTD_PUBLISHER_ID`). Some sit inline in the function definition — the
Gracenote recipes, for instance, put the API key in the ADS request URL because
that is what the API expects.

**Consider:**

- **Do not commit real keys.** Once you replace a placeholder with a live
  credential, that `function.json` contains a secret. Keep it out of source
  control, or inject the value at deploy time.
- **Prefer indirection where the integration supports it** — pass sensitive
  values as per-session parameters, or resolve them from a secrets manager,
  rather than hard-coding them into a stored function definition.
- **Scope and rotate.** Use least-privilege API keys and rotate them; a leaked
  ad-API key can enable quota abuse or ad fraud on your account.
- **Remember functions have no built-in versioning.** Keep your customized
  definitions (minus secrets) under your own source control so you can review
  and roll back.

## 2. Privacy, consent, and PII (GDPR / CCPA and similar)

Several recipes forward personal or device data to third parties:

- **LiveRamp ATS** sends a hashed email, client IP, and consent signals.
- **The Trade Desk OpenAds** recipes send client IP, device advertising ID
  (`ifa`), user-agent, and identity envelopes, with `regs`/consent fields.

**Consider:**

- **Consent is your responsibility, and the samples do not decide it for you.**
  The US-privacy default `us_privacy: "1---"` in the OpenAds samples is an
  IAB-conventional *placeholder meaning "no signal available"* — it is **not** a
  consent decision. For EU/UK traffic you must supply a valid TCF consent string
  from your CMP; for US traffic, populate `us_privacy` from your CMP.
- **Only send what you have a lawful basis to send.** Confirm your legal basis
  for each field forwarded to each third party in each jurisdiction you serve.
- **Honor `coppa`, `dnt`, and `lmt`.** The samples pass these through; make sure
  the values reflect reality for your audience and that downstream partners act
  on them.
- **Minimize.** Remove fields a given integration does not need.
- Consult each partner's privacy documentation
  ([LiveRamp](https://developers.liveramp.com/authenticatedtraffic-api),
  [The Trade Desk](https://open.thetradedesk.com/)) and your own DPO/legal
  guidance.

## 3. Untrusted input in requests

Recipes build outbound requests from player-supplied values.

**Consider:**

- **URL construction.** When a value is concatenated into a request **URL**,
  URL-encode it. The samples use `$encodeUrlComponent(...)` for this (for
  example around `content_title` and `content_tms_id`). If you add parameters,
  keep that discipline — an un-encoded, client-controlled value in a URL can
  alter the request.
- **Request bodies.** The OpenAds recipes build a JSON body with JSONata object
  construction and `$string(...)`, which handles JSON escaping. If you switch to
  hand-built body strings, you take on the escaping yourself.
- **Content-metadata source (asset recipes).** The `ttd-openads-video-asset-*`
  recipes read content metadata from `EXT-X-ASSET` manifest tags (`asset.*`).
  `EXT-X-ASSET` values are URL-encoded in the origin manifest; whether they are
  decoded before reaching `asset.*` depends on MediaTailor's handling and was
  **not verified** for these samples. Validate the values you actually receive
  and decode (`$decodeUrlComponent(...)`) if needed, and constrain them before
  forwarding.
- **Spoofable signals.** `session.client_ip` derives from `X-Forwarded-For`,
  which a client may influence upstream of MediaTailor. The LiveRamp recipe
  forwards it as `X-Forwarded-For` to the identity API. If you rely on IP for
  geo/identity, understand it can be spoofed and decide whether to trust it.

## 4. Fail-open behavior and availability

Monetization Functions **fail open**: if a function errors or times out,
MediaTailor proceeds with ad insertion *without* the function's output rather
than interrupting the stream. Recipes that call external services set a
`RequestTimeoutMilliseconds`.

**Consider:**

- **This is a deliberate trade-off you should make consciously.** A slow or
  unreachable third-party API means the ad request proceeds with missing or
  `null` enrichment — degraded targeting, not a broken stream.
- **Design your ADS URL and downstream logic to tolerate missing values.** The
  recipes guard on `response.statusCode = 200` and emit `null` otherwise; make
  sure your ad server handles empty/`null` parameters gracefully.
- **Tune timeouts** to balance enrichment completeness against added latency on
  every session/ad break.

## 5. Third-party trust

Each integration sends data to, and trusts responses from, an external service.

**Consider:**

- You are extending trust to each third party (LiveRamp, Gracenote, The Trade
  Desk) and to whatever their responses drive. Review their security and data
  handling.
- Validate response data before you act on it, especially anything that feeds
  back into a URL, body, or player parameter used later in the session.

## Reporting a problem

Found a security issue in these samples? See [SECURITY.md](../SECURITY.md) for
how to report it. For issues in your own deployed, customized functions, follow
your organization's process.
