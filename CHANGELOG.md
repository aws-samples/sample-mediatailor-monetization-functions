# Changelog

All notable changes to the published recipes and documentation in this
repository are recorded here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Categories used: **Added**, **Changed**, **Fixed**, **Removed**, **Security**.

## [Unreleased]

_Nothing yet. Add entries here as changes land; they will be stamped with the
version and date at the next release._

## [1.0.0] — 2026-08-20

Initial public release.

### Added

- **9 recipes** for AWS Elemental MediaTailor Monetization Functions, organized
  by the same two-axis (use case × service) taxonomy used in the MediaTailor
  console:
  - **Identity enrichment** — [LiveRamp ATS API](recipes/liveramp-ats-api/).
  - **Data processing** — [Gracenote Lookup by TMS ID](recipes/gracenote-lookup-by-tms-id/),
    [Gracenote Lookup by Title](recipes/gracenote-lookup-by-title/),
    [User-Agent Normalization](recipes/user-agent-normalization/).
  - **Ad decisioning** — four TTD OpenAds video variants covering US/EU
    consent regimes and Gracenote/asset-manifest metadata sources
    ([Gracenote US](recipes/ttd-openads-video-gracenote-us/),
    [Gracenote EU/UK](recipes/ttd-openads-video-gracenote-eu/),
    [Asset US](recipes/ttd-openads-video-asset-us/),
    [Asset EU/UK](recipes/ttd-openads-video-asset-eu/)).
  - **Traffic management** — [Random Traffic Split](recipes/random-traffic-split/).
- Every recipe is self-contained: metadata (`recipe.yaml`), the function
  definition (`function.json`), a `FunctionMapping` fragment (`mapping.json`),
  a per-recipe README with prerequisites and deploy steps, and a sample
  session-initialization request with the expected output under `test/`.
- Documentation:
  - [docs/concepts.md](docs/concepts.md) — lifecycle hooks, function types,
    output namespaces, and JSONata basics.
  - [docs/taxonomy.md](docs/taxonomy.md) — the use-case × service matrix.
  - [docs/security-considerations.md](docs/security-considerations.md) —
    customer-owned decisions around secrets, privacy/consent, untrusted input,
    fail-open behavior, and third-party trust.
- Standard repository files: `LICENSE` (MIT-0), `NOTICE`, `SECURITY.md`,
  `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, and a JSON Schema for `recipe.yaml`.

[Unreleased]: https://github.com/aws-samples/sample-mediatailor-monetization-functions/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/aws-samples/sample-mediatailor-monetization-functions/releases/tag/v1.0.0
