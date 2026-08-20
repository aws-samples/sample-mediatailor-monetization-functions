# AWS Elemental MediaTailor Monetization Functions — Recipe Library

Ready-to-use example **recipes** for [AWS Elemental MediaTailor Monetization
Functions](https://docs.aws.amazon.com/mediatailor/latest/ug/monetization-functions.html).

MediaTailor Monetization Functions let you customize how MediaTailor manages
session data and builds ad decision server (ADS) requests during ad insertion —
calling external APIs, transforming data, and modifying ADS request parameters,
with no infrastructure to deploy. Functions are written in
[JSONata](https://docs.aws.amazon.com/mediatailor/latest/ug/monetization-functions-jsonata.html)
and run at defined lifecycle hooks during playback.

This library mirrors the recipe templates available in the MediaTailor console
so you can copy, customize, deploy, and version-control them alongside your own
automation. Each recipe is self-contained and documented for both human readers
and automated indexing.

> **These are examples to learn from and customize — not production-ready code.**
> Review each recipe, and see [Security and privacy](#security-and-privacy),
> before deploying.

---

> **New to Monetization Functions?** Start with [docs/concepts.md](docs/concepts.md)
> for hooks, function types, and JSONata basics.

## Recipes

Recipes are organized by the same two-axis taxonomy used in the MediaTailor
console — **use case** and **service**. See [docs/taxonomy.md](docs/taxonomy.md)
for the full matrix and how to browse by either axis.

| Recipe | Use case | Service | Hook |
| --- | --- | --- | --- |
| [LiveRamp ATS API](recipes/liveramp-ats-api/) | Identity enrichment | LiveRamp | `PRE_SESSION_INITIALIZATION` |
| [Gracenote Lookup by TMS ID](recipes/gracenote-lookup-by-tms-id/) | Data processing | Gracenote | `PRE_ADS_REQUEST` |
| [Gracenote Lookup by Title](recipes/gracenote-lookup-by-title/) | Data processing | Gracenote | `PRE_ADS_REQUEST` |
| [TTD OpenAds Video — Gracenote (US)](recipes/ttd-openads-video-gracenote-us/) | Ad decisioning | The Trade Desk | `PRE_ADS_REQUEST` |
| [TTD OpenAds Video — Gracenote (EU/UK)](recipes/ttd-openads-video-gracenote-eu/) | Ad decisioning | The Trade Desk | `PRE_ADS_REQUEST` |
| [TTD OpenAds Video — Asset Metadata (US)](recipes/ttd-openads-video-asset-us/) | Ad decisioning | The Trade Desk | `PRE_ADS_REQUEST` |
| [TTD OpenAds Video — Asset Metadata (EU/UK)](recipes/ttd-openads-video-asset-eu/) | Ad decisioning | The Trade Desk | `PRE_ADS_REQUEST` |
| [Random Traffic Split](recipes/random-traffic-split/) | Traffic management · Ad decisioning | Custom Output | `PRE_ADS_REQUEST` |
| [User-Agent Normalization](recipes/user-agent-normalization/) | Data processing · Identity enrichment | Custom Output | `PRE_SESSION_INITIALIZATION` |

## Recipe layout

Every recipe folder is self-contained:

```text
recipes/<recipe-id>/
├── recipe.yaml    # Metadata: id, name, categories, services, hook, function type
├── function.json  # The PutFunction request body (the recipe itself)
├── mapping.json   # FunctionMapping fragment to attach the function to a playback config
├── README.md      # Scenario, prerequisites, values to replace, how to deploy
└── test/          # Sample session-init request + expected output
```

## Using a recipe

1. Open the recipe's `README.md` and replace every placeholder value it lists
   (for example, `YOUR_PLACEMENT_ID`).
2. Create the function from `function.json`.
3. Attach it to a playback configuration using `mapping.json`.
4. Reference any output values in your ADS URL via
   [dynamic variable substitution](https://docs.aws.amazon.com/mediatailor/latest/ug/variables-domain.html).

See each recipe's README for exact console and AWS CLI steps.

## Security and privacy

These recipes are **learning examples that you own and customize**, not
turnkey production functions. They favor simplicity and your freedom to adapt
them over prescriptive, one-size-fits-all controls. Before deploying any recipe,
review [docs/security-considerations.md](docs/security-considerations.md), which
covers the decisions the samples deliberately leave to you — secret handling,
privacy/consent (GDPR/CCPA), untrusted input, fail-open behavior, and
third-party trust.

To report a security issue in the samples, see [SECURITY.md](SECURITY.md).

## Contributing

Contributions are welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) and our
[Code of Conduct](CODE_OF_CONDUCT.md) before opening an issue or pull request.

## Changelog

Notable changes to the recipes and documentation are recorded in
[CHANGELOG.md](CHANGELOG.md).

## License

This library is licensed under the MIT-0 License. See [LICENSE](LICENSE).
