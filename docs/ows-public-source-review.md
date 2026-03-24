# OWS Public Source Review

This document records the public-source review used to revise this repository on **2026-03-24**.

## Scope

Every local analysis file was reread against public primary sources:

- `openwallet.sh`
- `docs.openwallet.sh`
- `github.com/open-wallet-standard/core`
- MoonPay public agents pages, Help Center articles, newsroom posts, and GitHub org pages

Unsupported claims were removed, downgraded to explicitly marked inference, or moved behind caveats.

## Authority Order

When public sources disagree, this repository now resolves them in this order:

1. numbered OWS docs in `open-wallet-standard/core/docs/`
2. `00-specification.md` and `08-conformance-and-security.md`
3. the public OWS README and `openwallet.sh`
4. MoonPay product docs for MoonPay-specific behavior only

## Main Findings

### 1. The public OWS sources are not fully aligned

The biggest examples found during review:

- the `docs.openwallet.sh` overview page uses higher-level request types that differ from the detailed `02-signing-interface.md`
- the overview page shows `Policy.action` as `"deny" | "warn"`, while `03-policy-engine.md` says the current policy-file format supports `action: "deny"` only
- the homepage describes policy checks using terms like allowlists and simulation requirements, while the detailed policy doc only standardizes `allowed_chains` and `expires_at` as built-in declarative rule types

This repository now follows the detailed numbered docs when those sources diverge.

### 2. The docs site navigation is incomplete

`docs.openwallet.sh` currently links the numbered docs through `07-supported-chains.md` in its top navigation. The public GitHub repo and README also expose `08-conformance-and-security.md`.

This repository now treats `08-conformance-and-security.md` as part of the public normative surface even though the docs-site nav currently omits it.

### 3. The quickstart should not be treated as the architecture authority

The public `quickstart.md` includes a `Signing Enclave (isolated proc)` diagram. The public `05-key-isolation.md` says current implementations are **in-process** and presents the subprocess enclave as a future profile.

This repository now treats `05-key-isolation.md` as the authoritative source for the current implementation posture.

### 4. The public Python import surface is inconsistent

The public `quickstart.md` uses:

```python
from open_wallet_standard import create_wallet, sign_message, sign_transaction
```

The public `sdk-python.md` uses:

```python
from ows import create_wallet, sign_message
```

This repository now documents both, but treats `sdk-python.md` as the more detailed source for the current Python API surface.

### 5. The README advertises a policy-engine implementation guide that is not directly accessible

The public OWS README lists a `Policy Engine Implementation Guide` in the reference-implementation docs list. During this review, that file was not directly accessible from the public docs tree or docs site.

This repository therefore removed module-level claims that depended on that guide and now treats it as an advertised-but-not-currently-accessible public document.

### 6. MoonPay product docs and OWS reference docs should not be conflated

The public OWS reference implementation docs describe a `~/.ows/` vault with AES-256-GCM and scrypt / HKDF-based envelopes.

MoonPay Help Center docs describe MoonPay CLI wallets as local HD wallets with **OS keychain encryption**, while separately saying MoonPay Agents implements OWS.

The safe reading is:

- MoonPay positions its product as implementing OWS
- MoonPay product docs are not proof that the MoonPay product uses the exact same storage layout as the public OWS reference implementation

### 7. The chain-spec doc and current SDK examples differ on the visible default account set

The numbered `07-supported-chains.md` defines **9 chain families** and includes Spark.

The current public CLI, Node, and Python examples for automatically derived accounts show **8** chain accounts and do not include Spark in those example outputs.

This repository now treats:

- `07-supported-chains.md` as the normative supported-family set
- the CLI / SDK docs as the current example implementation surface

## Coverage Matrix

| Public source | Covered by local docs |
|---|---|
| `openwallet.sh` homepage | `ows-overview.md`, `ows-architecture.md`, `ows-x402-payments-integration.md` |
| `docs/00-specification.md` | `ows-specification-conformance-and-security.md` |
| `docs/01-storage-format.md` | `ows-storage-format.md` |
| `docs/02-signing-interface.md` | `ows-signing-interface.md` |
| `docs/03-policy-engine.md` | `ows-policy-engine.md` |
| `docs/04-agent-access-layer.md` | `ows-agent-access-layer.md`, `ows-architecture.md` |
| `docs/05-key-isolation.md` | `ows-key-isolation-and-security.md`, `ows-architecture.md` |
| `docs/06-wallet-lifecycle.md` | `ows-wallet-lifecycle.md` |
| `docs/07-supported-chains.md` | `ows-supported-chains.md` |
| `docs/08-conformance-and-security.md` | `ows-specification-conformance-and-security.md` |
| `docs/quickstart.md` | `ows-reference-implementation-guide.md`, `ows-sdk-cli-reference.md` |
| `docs/sdk-cli.md` | `ows-sdk-cli-reference.md`, `ows-x402-payments-integration.md` |
| `docs/sdk-node.md` | `ows-sdk-cli-reference.md` |
| `docs/sdk-python.md` | `ows-sdk-cli-reference.md`, `ows-reference-implementation-guide.md` |
| `open-wallet-standard/core` README | `ows-overview.md`, `ows-architecture.md`, `ows-github-repos.md`, `ows-sdk-cli-reference.md` |
| `www.moonpay.com/agents` | `ows-overview.md`, `ows-moonpay-agents-and-skills.md` |
| MoonPay Help Center AI agents / CLI / MCP articles | `ows-moonpay-agents-and-skills.md`, `ows-x402-payments-integration.md` |
| MoonPay GitHub org page | `ows-github-repos.md`, `ows-moonpay-agents-and-skills.md` |

## Changes Made In This Review

- removed unsupported authorship, launch-date, and repo-structure claims
- removed or softened path-level claims about `moonpay/skills` that were not directly verifiable from public sources used here
- downgraded the policy-engine implementation guide from a relied-on source to an advertised-but-not-directly-accessible doc
- added missing storage, lifecycle, policy, and SDK details that are present in the public numbered docs or public SDK docs
- added explicit notes wherever public OWS overview pages drift from the more detailed numbered docs

## Primary Sources

- `https://openwallet.sh/`
- `https://docs.openwallet.sh/`
- `https://github.com/open-wallet-standard/core`
- `https://www.moonpay.com/agents`
- `https://support.moonpay.com/en/articles/586487-moonpay-agents-fund-your-ai`
- `https://support.moonpay.com/en/articles/586583-moonpay-cli-for-ai-agents`
- `https://support.moonpay.com/en/articles/592667-connect-moonpay-to-any-mcp-compatible-ai`
- `https://support.moonpay.com/en/articles/592627-openclaw-moonpay-cli-setup-guide`
- `https://www.moonpay.com/it/newsroom/moonpay-agents`
- `https://github.com/moonpay`
