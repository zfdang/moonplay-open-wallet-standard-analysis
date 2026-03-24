# Open Wallet Standard (OWS) Overview

## What OWS Is

The Open Wallet Standard (OWS) is an open specification for local wallet storage, delegated agent access, and policy-gated signing. It defines how cryptographic wallets are created, stored on a local filesystem, and accessed by AI agents and CLI tools through a unified interface.

OWS was created by **MoonPay** and launched on **March 23, 2026** as v1.0.0. It is the wallet layer for MoonPay's agent economy initiative ("MoonPay Agents").

The core idea: every CLI tool, AI agent, and automation script currently invents its own key management — env vars with raw private keys, proprietary cloud APIs, bespoke keystore formats. OWS replaces this fragmentation with a single open standard.

## Core Design Principles

OWS is built around six guiding principles:

### 1. Local-First

Keys live in `~/.ows/` — not in a browser extension, not in the cloud, not scattered across tool-specific config directories. Everything operates on the local filesystem.

### 2. No API Calls

No HTTP. No vendor lock-in. No authentication flows. No rate limits. OWS runs entirely on the user's machine. The signing core is an in-process library, not a remote service.

### 3. Multi-Chain by Default

One wallet, every chain. A single BIP-39 mnemonic derives accounts for BTC, ETH, SOL, ATOM, TON, TRON, SUI, Spark, Filecoin — all via standard BIP-44 derivation paths. Chain-agnostic identifiers (CAIP-2 / CAIP-10) are used throughout.

### 4. Self-Custody

Your keys. Your device. No remote signing. No custodians. No third-party access. The mnemonic is encrypted at rest with AES-256-GCM and never leaves the local vault.

### 5. Zero-Trust Agent Access

Agents never see plaintext keys. Instead, they authenticate with scoped API tokens (`ows_key_...`). The token serves as both an authentication credential and decryption capability. Policies are enforced before any key material is touched.

### 6. Composable

Works with any tool that speaks JSON. CLI, MCP, SDK, REST — same wallet, same security model. A wallet created by one tool works in every other conforming implementation.

## The Problem OWS Solves

Today, every tool reinvents the wallet:

| Tool | Key Location | Encryption |
|------|-------------|------------|
| Foundry | `~/.foundry/keystores/` | Keystore v3 |
| Hardhat | `.env PRIVATE_KEY=0xab3f...` | None (plaintext) |
| Solana CLI | `~/.config/solana/id.json` | None (plaintext JSON) |
| Agent X | `.env.local SIGNER=0x91c...` | None (plaintext) |
| Custom bot | `./config/wallet.json` | Varies |
| Shell history | `export PRIVATE_KEY=0x...` | None (plaintext) |

This is **6 formats, 6 locations, 0 encryption** in many cases.

With OWS: **1 vault (`~/.ows/`), 1 interface, AES-256-GCM encrypted**.

## Who OWS Is For

- **AI agent developers** who need their agents to sign transactions without exposing private keys
- **CLI tool builders** who want a shared, encrypted wallet that works across tools
- **Protocol developers** who need a standardized local signing interface
- **dApp developers** who want to give agents scoped, policy-controlled wallet access

## Relationship to MoonPay Agents

MoonPay Agents is MoonPay's product line for the "agent economy." It includes:

- The MoonPay CLI (`npm i -g @moonpay/cli`)
- A skill library (30+ skills for wallet management, token trading, cross-chain bridges, prediction markets, etc.)
- MCP server integration
- Integration with Claude, Cursor, Windsurf, Codex, and 40+ other AI agents

OWS is the **wallet layer** that MoonPay Agents implements. The announcement states: "MoonPay Agents implements the Open Wallet Standard — an open standard for agent-to-wallet communication."

However, OWS is intentionally separate from MoonPay as a product. It is MIT-licensed and designed to be adopted by any tool or framework.

## Ecosystem Participants

At launch, the OWS website lists logos and partnerships with:

**Blockchain Foundations & Networks:** PayPal, Ripple, OKX, Arbitrum, TRON, Solana Foundation, Base, Circle, Ethereum Foundation, Polygon, Sui, Filecoin Foundation, TON (The Open Network Foundation)

**Infrastructure & Protocols:** LayerZero, Virtual Protocol, Uniblock, DFlow, DFNS, Dynamic, Allium, Simmer

## Inspired By

OWS explicitly draws from:

| Project | What OWS Borrowed |
|---------|-------------------|
| x402 | Spec structure, scheme/network/transport separation |
| Privy | Policy engine design, key sharding concepts, CAIP-2 chain identifiers |
| Coinbase AgentKit | ActionProvider/WalletProvider pattern, MCP tool exposure |
| Keystore v3 | Proven encrypted storage format since 2015 |
| CAIP Standards | Chain-agnostic identifiers for chains, accounts, and methods |
| ERC-4337 | Session keys, programmable validation, paymaster sponsorship |
| Turnkey | TEE-based signing, sub-100ms latency targets |
| W3C Universal Wallet | lock/unlock/import/export interface patterns |
| Solana Wallet Standard | Feature-based capability registration |
| Crossmint | Dual-key model, on-chain policy enforcement |
| Lit Protocol | Decentralized key management, IPFS-published policies |
| WalletConnect v2 | Session authorization model, relay architecture |

OWS states: "No new primitives. OWS doesn't invent new cryptography or chain-specific abstractions. It implements existing standards in a composable, agent-friendly way."
