# MoonPay Agents And Skills Ecosystem

How MoonPay publicly positions OWS inside MoonPay Agents, MoonPay CLI, MCP, and the published MoonPay skill surface.

## What The Public Sources Establish

From MoonPay's agents page, Help Center articles, newsroom post, and GitHub org page, the directly supported points are:

- MoonPay Agents publicly says it **implements the Open Wallet Standard**
- MoonPay Agents is described as a **non-custodial software layer**
- MoonPay distributes the CLI package `@moonpay/cli`
- MoonPay CLI can run as a local MCP server via `mp mcp`
- MoonPay's Help Center describes MoonPay Agents as exposing **54 crypto-specific tools across 17 skills**
- the MoonPay GitHub org publicly includes a `moonpay/skills` repository

## MoonPay Agents

The agents landing page presents MoonPay Agents as:

- terminal-first
- authenticated
- wallet-enabled
- ready to transact

The public marketing page says:

> MoonPay Agents implements the Open Wallet Standard

The newsroom launch post says MoonPay Agents was published on **February 24, 2026** and describes it as a non-custodial software layer that gives AI agents access to wallets, funds, and autonomous transaction capability through MoonPay CLI.

One public-source nuance is worth keeping in mind:

- the landing page markets "20+ skills"
- the Help Center article explicitly enumerates **17 skills / 54 tools**

This repository follows the Help Center article when it needs exact counts or names.

## MoonPay CLI

MoonPay Help Center describes MoonPay CLI as offering:

- non-custodial local wallets
- swaps, bridges, and transfers
- market data
- fiat on/off-ramp flows
- virtual accounts
- x402-related flows

Publicly documented access surfaces are:

- CLI: `mp`
- MCP server: `mp mcp`
- REST API
- web chat

## Supported Chains In MoonPay CLI

MoonPay Help Center currently lists:

- Solana
- Ethereum
- Base
- Polygon
- Arbitrum
- Optimism
- BNB
- Avalanche
- TRON
- Bitcoin

That is MoonPay product behavior, not the OWS reference-chain matrix.

## Published MoonPay Skill Names

MoonPay Help Center explicitly lists these 17 skills:

- `moonpay-auth`
- `moonpay-block-explorer`
- `moonpay-buy-crypto`
- `moonpay-check-wallet`
- `moonpay-deposit`
- `moonpay-export-data`
- `moonpay-feedback`
- `moonpay-fund-polymarket`
- `moonpay-mcp`
- `moonpay-missions`
- `moonpay-price-alerts`
- `moonpay-swap-tokens`
- `moonpay-trading-automation`
- `moonpay-upgrade`
- `moonpay-virtual-account`
- `moonpay-x402`
- `moonpay-discover-tokens`

The same Help Center material also says these 17 skills expose **54 tools** in total.

## GitHub Presence

The MoonPay GitHub org page shows a public `skills` repository and describes it as:

- "Skills for AI agents to move money — on-ramps, swaps, wallets, deposits, and more via the MoonPay CLI"

That is enough to describe the repo's public role. This repository no longer relies on unverified path-level claims inside that repo.

## MCP And Client Integration

MoonPay's MCP setup article documents `mp mcp` and shows configuration examples for:

- Claude Code
- Claude Desktop
- OpenClaw

MoonPay's Help Center also mentions install shortcuts for:

- Claude
- ChatGPT
- Gemini
- Grok

The important distinction is:

- those are MoonPay product integration docs
- they do not redefine the OWS standard itself

## Security And Storage Boundary

MoonPay Help Center says MoonPay CLI uses local HD wallets with **OS keychain encryption** and that keys never leave the machine.

That should be kept distinct from the public OWS reference docs, which describe:

- a `~/.ows/` vault
- AES-256-GCM envelopes
- scrypt / HKDF-based local artifact formats

The safe synthesis is:

- MoonPay positions its product as non-custodial and OWS-based
- MoonPay product docs are not proof that MoonPay uses the exact same file layout or packaging as the public OWS reference implementation

## Relationship Between OWS And MoonPay

| Topic | OWS | MoonPay Agents / CLI |
|---|---|---|
| Nature | Open wallet standard | MoonPay product and integration surface |
| Public source type | Spec docs, README, reference-implementation docs | Agents page, Help Center, newsroom, GitHub org |
| Scope | Storage, signing, policy, lifecycle, chains, conformance | Wallet UX, MCP, swaps, deposits, fiat, x402, skills |
| Storage description | `~/.ows/` local vault | local wallets with OS keychain encryption |
| Standard status | defines interoperability expectations | one public implementation / product layer |

## Primary Sources

- `https://www.moonpay.com/agents`
- `https://support.moonpay.com/en/articles/586487-moonpay-agents-fund-your-ai`
- `https://support.moonpay.com/en/articles/586583-moonpay-cli-for-ai-agents`
- `https://support.moonpay.com/en/articles/592667-connect-moonpay-to-any-mcp-compatible-ai`
- `https://support.moonpay.com/en/articles/592627-openclaw-moonpay-cli-setup-guide`
- `https://www.moonpay.com/it/newsroom/moonpay-agents`
- `https://github.com/moonpay`
