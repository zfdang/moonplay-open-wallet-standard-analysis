# OWS Reference Implementation Guide

This document covers the **public non-normative implementation docs** that complement the numbered OWS specification.

## What Is Directly Accessible

The directly accessible public reference-implementation docs used in this review are:

- `quickstart.md`
- `sdk-cli.md`
- `sdk-node.md`
- `sdk-python.md`

The public OWS README also advertises a `Policy Engine Implementation Guide`, but that guide was **not directly accessible** during this review. This repository therefore no longer treats it as a stable source for module-level claims.

## How To Read These Docs Correctly

These docs are useful for:

- installer behavior
- package names
- CLI syntax
- SDK examples
- current implementation ergonomics

They are **not** the interoperability standard. If they conflict with the numbered docs, the numbered docs win.

## Quickstart

The public quickstart is explicitly labeled non-normative. It covers:

- installation
- wallet creation
- wallet funding
- x402-powered paid requests
- signing
- basic Node.js and Python examples
- API-key-based agent access

It is the best public onboarding document, but it should not be used as the final authority for architecture or interface details.

## CLI Reference

`sdk-cli.md` is the detailed operational reference for the public `ows` binary. It documents:

- wallet commands
- policy commands
- key commands
- signing commands
- mnemonic commands
- payment commands
- funding commands
- update / uninstall commands
- the current `~/.ows/` file layout used by the reference implementation

This repository now treats `sdk-cli.md` as the main source for the current OWS CLI surface.

## Node.js SDK

`sdk-node.md` documents:

- the package name `@open-wallet-standard/core`
- native bindings via NAPI
- in-process Rust execution
- function-based APIs rather than a client object
- default vault root behavior and optional `vaultPath`

This is the main public source for the current Node.js binding surface.

## Python SDK

`sdk-python.md` documents:

- the package name `open-wallet-standard`
- native bindings via PyO3
- in-process Rust execution
- function-based APIs
- default vault root behavior and optional `vault_path`

This is the main public source for the current Python binding surface.

## Important Public-Doc Drift

Two public inconsistencies matter enough to call out here:

### Quickstart versus key-isolation docs

The quickstart shows a `Signing Enclave (isolated proc)` in its architecture diagram. The public `05-key-isolation.md` says current implementations are **in-process** and describes subprocess enclave isolation as a future profile.

For architecture claims, this repository now follows `05-key-isolation.md`.

### Quickstart versus Python SDK docs

The quickstart uses:

```python
from open_wallet_standard import create_wallet, sign_message, sign_transaction
```

The public Python SDK page uses:

```python
from ows import create_wallet, sign_message
```

This repository now documents both, but treats `sdk-python.md` as the more detailed source for the current Python API shape.

## What This Means For This Repository

The clean reading order is:

1. numbered docs for standard behavior
2. conformance/security docs for claims and minimum requirements
3. quickstart and SDK docs for packaging and usage
4. MoonPay docs for MoonPay-specific product behavior

That separation prevents the most common mistake in earlier drafts of this repository: turning current installer or SDK ergonomics into claims about the standard itself.

## Primary Sources

- `https://github.com/open-wallet-standard/core`
- `https://github.com/open-wallet-standard/core/blob/main/docs/quickstart.md`
- `https://github.com/open-wallet-standard/core/blob/main/docs/sdk-cli.md`
- `https://raw.githubusercontent.com/open-wallet-standard/core/main/docs/sdk-node.md`
- `https://github.com/open-wallet-standard/core/blob/main/docs/sdk-python.md`
