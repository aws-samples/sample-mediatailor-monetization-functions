# Security Policy

## Reporting a vulnerability

If you discover a potential security issue in these sample recipes, we ask that
you notify AWS Security via our
[vulnerability reporting page](https://aws.amazon.com/security/vulnerability-reporting/)
or directly via email to <aws-security@amazon.com>. Please do **not** create a
public GitHub issue for security reports.

## Scope

This repository contains **example recipes** for AWS Elemental MediaTailor
Monetization Functions. The samples are intended to be copied and customized;
they are not a running service.

- Reports about the **sample recipes or repository tooling** in this project are
  welcome via the channel above.
- Issues in **your own deployed, customized functions** are outside this
  repository's scope — follow your organization's security process, and see
  [docs/security-considerations.md](docs/security-considerations.md) for the
  decisions these samples deliberately leave to you.

## Using these samples securely

Before deploying any recipe, review
[docs/security-considerations.md](docs/security-considerations.md). It covers
secret handling, privacy/consent obligations, untrusted-input handling,
fail-open behavior, and third-party trust — the areas you own when you adapt a
sample for production.
