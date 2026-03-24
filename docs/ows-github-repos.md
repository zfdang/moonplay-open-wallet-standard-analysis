# OWS GitHub Repositories

What is publicly visible today in the OWS and MoonPay GitHub surface, and what this repository deliberately avoids claiming without a directly accessible source.

## Primary OWS Repository

The canonical public OWS repository is:

- [github.com/open-wallet-standard/core](https://github.com/open-wallet-standard/core)

The public repo page presents it as both:

- the public specification repository
- the public reference-implementation repository

## What Is Directly Visible From The Repo Root

The top-level tree visible on the public repo page includes:

- `.githooks/`
- `.github/`
- `bindings/`
- `docs/`
- `ows/`
- `readme/`
- `scripts/`
- `skills/ows/`
- `website-docs/`
- `README.md`
- `LICENSE`
- `SECURITY.md`
- `CONTRIBUTING.md`
- `CHANGELOG.md`

That tree is safe to describe because it is visible directly on the public repo root page.

## Public Documentation Surface

The public sources show three overlapping documentation surfaces:

1. the GitHub `docs/` directory
2. `docs.openwallet.sh`
3. the repo README's spec / reference-doc list

Across those sources, the publicly advertised docs include:

- `00-specification.md`
- `01-storage-format.md`
- `02-signing-interface.md`
- `03-policy-engine.md`
- `04-agent-access-layer.md`
- `05-key-isolation.md`
- `06-wallet-lifecycle.md`
- `07-supported-chains.md`
- `08-conformance-and-security.md`
- `quickstart.md`
- `sdk-cli.md`
- `sdk-node.md`
- `sdk-python.md`

## Documentation Drift Worth Noting

Two review findings matter here:

- `docs.openwallet.sh` currently exposes the numbered docs through `07-supported-chains.md` in its top navigation, while the repo README also includes `08-conformance-and-security.md`
- the repo README advertises a `Policy Engine Implementation Guide`, but that guide was not directly accessible during this review

This repository therefore treats:

- `08-conformance-and-security.md` as public and important
- the policy-engine implementation guide as an advertised reference doc, not as a stable directly reviewable source

## Bindings And Packaging

The public repo root directly shows:

- `bindings/`
- `ows/`
- `skills/ows/`
- `website-docs/`

The public README and SDK docs also make these claims safely:

- Node.js is published as `@open-wallet-standard/core`
- Python is published as `open-wallet-standard`
- the CLI binary is `ows`
- the language bindings embed the Rust core through native FFI

What this repository no longer claims without a directly accessible file:

- a precise Rust crate-by-crate or module-by-module breakdown beyond what is visible in the public docs used here
- specific internal source paths for MoonPay's `skills` repo

## MoonPay GitHub Organization

The public MoonPay org page shows **4 public repositories** at the time of review:

- `skills`
- `devops-challenge`
- `moonpay-demo-integrations`
- `moonpay-sign`

For OWS analysis, the only repo that matters directly is:

- `moonpay/skills`

The org page describes it as:

- "Skills for AI agents to move money — on-ramps, swaps, wallets, deposits, and more via the MoonPay CLI"

The org page also shows it as public, MIT-licensed, and updated on **March 23, 2026**.

## What This Repository Avoids Claiming About `moonpay/skills`

Earlier drafts in this repository included path-level claims about the internal layout of `moonpay/skills`. During this review, those claims were removed unless they could be tied to a directly accessible public source used here.

The safe GitHub-level statements are:

- the repo exists publicly under the MoonPay org
- MoonPay describes it as a skills repo for AI agents and MoonPay CLI workflows
- MoonPay's Help Center articles list the public skill names and tool families

## Build And Contribution Surface

The public OWS repo clearly exposes:

- `README.md`
- `LICENSE`
- `SECURITY.md`
- `CONTRIBUTING.md`
- releases

The public README also documents:

```bash
git clone https://github.com/open-wallet-standard/core.git
cd core/ows
cargo build --workspace --release
```

That is enough to safely describe OWS as a public open-source spec-plus-reference-implementation project with source-build instructions.

## Takeaway

The clean public boundary is:

- `open-wallet-standard/core` = the OWS spec and its public reference-implementation surface
- MoonPay public web / help-center / org pages = MoonPay's product and ecosystem layer around agents, CLI, skills, and payments

## Primary Sources

- `https://github.com/open-wallet-standard/core`
- `https://docs.openwallet.sh/`
- `https://openwallet.sh/`
- `https://github.com/moonpay`
- `https://support.moonpay.com/en/articles/586487-moonpay-agents-fund-your-ai`
