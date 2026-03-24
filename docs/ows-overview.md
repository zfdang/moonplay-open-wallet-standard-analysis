# Open Wallet Standard (OWS) Overview

## What OWS Is

The public OWS docs describe OWS as an open standard for:

- local wallet storage
- delegated agent access
- policy-gated signing

The OWS homepage frames the problem as fragmented local key management across CLIs, agents, and scripts, and presents OWS as a shared local storage and signing standard rather than a hosted wallet API.

The safest summary is:

- OWS standardizes local wallet artifacts and signing behavior
- OWS is designed for agents, CLIs, and other local tools
- OWS is multi-chain and uses CAIP identifiers across the spec

## What Can Be Stated Safely About Ownership And Release

The public OWS pages reviewed here clearly show **v1.0.0** and a public GitHub repo, but they do not need an authorship claim to explain the standard.

The public MoonPay agents page separately states that **MoonPay Agents implements the Open Wallet Standard**, which establishes a product relationship. This repository therefore avoids stronger statements like "MoonPay created OWS" unless a primary OWS source says so directly.

## Core Design Principles

The OWS homepage presents six principles:

### 1. Local-first

Keys live in `~/.ows/` rather than in cloud custody or tool-specific directories.

### 2. No API calls

The homepage describes OWS as running entirely on the local machine, without HTTP or vendor-auth flows in the core wallet path.

### 3. Multi-chain by default

One wallet derives accounts across the supported chain families from a single seed.

### 4. Self-custody

The user controls the keys and device. The model is explicitly non-custodial.

### 5. Zero-trust agent access

Agents authenticate with scoped API tokens instead of seeing plaintext keys.

### 6. Composable

The homepage presents the same wallet and security model across CLI, SDK, MCP, and REST-style access surfaces.

## The Problem OWS Solves

The homepage contrasts today's fragmented local-wallet patterns with the OWS vault:

| Example | Public homepage example |
|---|---|
| Foundry | `~/.foundry/keystores/` |
| Hardhat | `.env PRIVATE_KEY=...` |
| Solana CLI | `~/.config/solana/id.json` |
| Custom agent or bot | env vars or local JSON files |
| Shell history | `export PRIVATE_KEY=...` |

The homepage's punchline is the move from many tool-specific locations and plaintext-key patterns to a single encrypted local vault.

## Relationship To MoonPay Agents

The MoonPay agents page says:

> MoonPay Agents implements the Open Wallet Standard

MoonPay's newsroom and Help Center describe MoonPay Agents as a non-custodial software layer built around MoonPay CLI. Those public MoonPay sources also describe MoonPay CLI as exposing wallets, swaps, bridges, deposits, fiat on/off-ramps, MCP access, and x402-related flows.

The important boundary is:

- OWS defines the open wallet standard
- MoonPay Agents is one public product that says it implements that standard
- MoonPay product docs should not be treated as proof of the exact OWS reference-implementation storage layout or packaging

## Ecosystem Signals On The Homepage

The OWS homepage displays logos from a broad set of networks and ecosystem companies, including PayPal, Ripple, OKX, Arbitrum, TRON, Solana Foundation, Base, Circle, Ethereum Foundation, Polygon, Sui, Filecoin Foundation, LayerZero, DFNS, and others.

The safe interpretation is that the homepage is signaling ecosystem relevance and intended reach. The logo wall alone is **not** a precise technical claim about implementation status, conformance, or formal partnership terms.

## Inspired By

The homepage explicitly lists these prior-art influences:

| Project | Publicly stated influence |
|---|---|
| x402 | Spec structure, scheme/network/transport separation, contribution templates |
| Privy | Policy engine design, key sharding concepts, CAIP-2 identifiers |
| Coinbase AgentKit | ActionProvider / WalletProvider pattern, MCP tool exposure |
| Keystore v3 | Encrypted storage format precedent |
| CAIP standards | Chain, account, and method identifiers |
| ERC-4337 | Session keys and programmable validation ideas |
| Turnkey | TEE-based signing inspiration |
| W3C Universal Wallet | Lock / unlock / import / export interface patterns |
| Solana Wallet Standard | Capability-registration ideas |
| Crossmint | Dual-key and policy concepts |
| Lit Protocol | Decentralized key-management ideas |
| WalletConnect v2 | Session authorization and relay concepts |

The homepage also says OWS is not inventing new cryptographic primitives; it is presenting a composable, agent-friendly packaging of existing standards.

## Primary Sources

- `https://openwallet.sh/`
- `https://docs.openwallet.sh/`
- `https://github.com/open-wallet-standard/core`
- `https://www.moonpay.com/agents`
- `https://www.moonpay.com/it/newsroom/moonpay-agents`
