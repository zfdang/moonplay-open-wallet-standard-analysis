# OWS x402 Payments Integration

What the public OWS and MoonPay sources do and do not establish about x402 payment flows.

## What Is Directly Supported By Public OWS Sources

The public OWS materials support these points:

- the OWS homepage lists **x402** in the "Inspired by" section
- the public quickstart describes `ows pay request` and `ows pay discover`
- the public CLI reference documents `ows pay request` and `ows pay discover`
- the CLI reference says `ows pay request` handles a `402 Payment Required` response by signing an **EIP-3009 `TransferWithAuthorization` for USDC** and retrying
- the CLI reference says `ows pay discover` discovers x402-enabled services from a **Bazaar** directory

These are public reference-implementation capabilities.

## What The Core Specification Says

`00-specification.md` explicitly places **on-chain service payment flows** out of scope for the core OWS specification.

That means:

- x402 is relevant to the public OWS toolchain
- x402 is **not** part of the core conformance contract for storage, signing, policy, lifecycle, or chain support

## What Can Be Said Safely About Bazaar Discovery

The public CLI docs let us say:

- `ows pay discover` exists
- it uses a Bazaar directory
- it supports client-side filtering and pagination

The public docs do **not** give enough basis to standardize:

- Bazaar registry schema
- service registration protocol
- response-body contract beyond the CLI examples

So this repository now treats Bazaar as a public **reference-implementation discovery surface**, not as a normative OWS subsystem.

## Relationship To OWS Wallet Semantics

When x402 flows use an OWS wallet, the public OWS docs still require:

- wallet lookup
- credential handling
- policy evaluation before token-backed secret decryption
- chain-specific signing
- audit logging without secret leakage

So x402 sits **on top of** OWS signing and policy behavior rather than replacing it.

## MoonPay's Public x402 Surface

MoonPay Help Center documents two x402-related MoonPay CLI surfaces:

- `x402 request`
- `upgrade`

The `upgrade` flow is documented as an x402 payment to increase API rate limits, settled in USDC on Solana or Base.

MoonPay also publishes the skill name:

- `moonpay-x402`

Those are MoonPay product features, not OWS core-spec requirements.

## What Was Removed In This Review

Earlier drafts in this repository mentioned third-party x402 skills and partner-marketplace details that were not directly verifiable from the public primary sources used in this review.

This file now limits itself to:

- public OWS docs
- public MoonPay docs

## Takeaway

x402 is best understood as:

- an explicit inspiration for OWS
- a documented capability in the public OWS reference implementation
- a documented capability in MoonPay CLI
- **not** part of the OWS core interoperability contract

## Primary Sources

- `https://openwallet.sh/`
- `https://github.com/open-wallet-standard/core/blob/main/docs/00-specification.md`
- `https://github.com/open-wallet-standard/core/blob/main/docs/quickstart.md`
- `https://github.com/open-wallet-standard/core/blob/main/docs/sdk-cli.md`
- `https://support.moonpay.com/en/articles/586583-moonpay-cli-for-ai-agents`
- `https://www.moonpay.com/agents`
