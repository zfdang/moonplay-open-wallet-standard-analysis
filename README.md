# MoonPay Open Wallet Standard — Analysis

This repository contains a comprehensive technical analysis of the **Open Wallet Standard (OWS)** and the MoonPay Agents ecosystem built on top of it.

```
Open Wallet Standard (OWS)
├── Storage Layer
│   ├── Vault directory structure (~/.ows/vault/)
│   ├── AES-256-GCM encrypted wallet files
│   ├── Scrypt KDF (passphrase → key)
│   ├── HKDF-SHA256 (API key → key)
│   └── Audit log
├── Signing Interface
│   ├── sign (raw transaction)
│   ├── signAndSend (sign + broadcast)
│   ├── signMessage (EIP-191, Solana off-chain, etc.)
│   └── signTypedData (EIP-712)
├── Policy Engine
│   ├── Declarative rules (allowed_chains, expires_at)
│   ├── Custom executable policies (stdin/stdout)
│   └── Default-deny on timeout or error
├── Agent Access Layer
│   ├── Owner mode (passphrase)
│   ├── Agent mode (API key — scoped, revocable)
│   └── Access profiles (in-process, subprocess, local service)
├── Key Isolation & Security
│   ├── mlock / zeroize / anti-ptrace
│   ├── Key caching with TTL
│   └── Future: subprocess enclave model
├── Wallet Lifecycle
│   ├── Create / Import / Export / Delete
│   ├── Backup & Recovery
│   └── Key Rotation
├── Supported Chains (9 families)
│   ├── EVM (Ethereum, Base, Polygon, Arbitrum, Optimism, BSC, Avalanche)
│   ├── Solana, Bitcoin, Cosmos, Tron, TON, Sui, Spark, Filecoin
│   └── BIP-32/39/44 derivation · CAIP-2/CAIP-10 identifiers
├── SDK & CLI
│   ├── Rust core (ows/)
│   ├── Node.js NAPI binding (@open-wallet-standard/core)
│   ├── Python CFFI binding (open-wallet-standard)
│   └── CLI (ows-cli)
├── x402 Payments
│   ├── ows pay request / ows pay discover
│   └── Bazaar directory
└── MoonPay Agents (flagship implementation)
    ├── Skills framework (30+ skills)
    ├── MCP server (Claude, Cursor, Windsurf)
    ├── On/off-ramp (buy/sell crypto with fiat)
    └── MoonPay CLI (@moonpay/cli)
```

## Recommended Reading Order

### Context and Overview

1. [ows-overview.md](docs/ows-overview.md)  
   What OWS is, what problem it solves, who it's for, the six core design principles, and the relationship to MoonPay Agents.

2. [ows-architecture.md](docs/ows-architecture.md)  
   Layered architecture diagrams, request flow for owner and agent signing, access model, technology stack, and the three access profiles.

### Storage and Cryptography

3. [ows-storage-format.md](docs/ows-storage-format.md)  
   Vault directory structure, wallet file format, API key file format, cryptographic parameters (AES-256-GCM, scrypt, HKDF), audit log, and backward compatibility with Keystore v3.

4. [ows-key-isolation-and-security.md](docs/ows-key-isolation-and-security.md)  
   In-process key lifecycle, memory hardening (mlock, zeroize, anti-ptrace), passphrase handling modes, threat model, and the future subprocess enclave model.

### Signing and Policies

5. [ows-signing-interface.md](docs/ows-signing-interface.md)  
   The four signing operations (sign, signAndSend, signMessage, signTypedData), TypeScript type definitions, chain-specific conventions, error codes, and concurrency.

6. [ows-policy-engine.md](docs/ows-policy-engine.md)  
   Owner vs agent access, API key cryptography (HKDF derivation, token-as-capability), declarative rules, custom executable policies (stdin/stdout protocol), evaluation order, default-deny semantics.

### Access and Lifecycle

7. [ows-agent-access-layer.md](docs/ows-agent-access-layer.md)  
   How agents interact with OWS wallets: required operations, credential semantics, three access profiles (in-process, subprocess, local service), cross-layer consistency.

8. [ows-wallet-lifecycle.md](docs/ows-wallet-lifecycle.md)  
   Wallet creation, import (5 formats), export (3 formats), backup and recovery, secure deletion, key rotation, wallet discovery, and lifecycle state diagram.

### Chain Support

9. [ows-supported-chains.md](docs/ows-supported-chains.md)  
   Nine chain families, CAIP-2/CAIP-10 identifiers, BIP-44 derivation paths, known networks, shorthand aliases, HD derivation tree, and how to add a new chain.

### SDK, CLI, and Ecosystem

10. [ows-sdk-cli-reference.md](docs/ows-sdk-cli-reference.md)  
    CLI commands, Node.js SDK API, Python SDK API, installation methods, environment variables, and the MoonPay CLI wrapper.

11. [ows-moonpay-agents-and-skills.md](docs/ows-moonpay-agents-and-skills.md)  
    MoonPay Agents product, skills framework (core, trading, on/off-ramp, research), MCP server integration with Claude/Cursor, agent security model, and the OWS vs MoonPay distinction.

12. [ows-x402-payments-integration.md](docs/ows-x402-payments-integration.md)  
    The x402 payment protocol, agent-to-service payment flow, Bazaar directory, policy integration for spending limits, and security considerations.

### Repository Guide

13. [ows-github-repos.md](docs/ows-github-repos.md)  
    Guide to the `open-wallet-standard/core` repository structure, `moonpay/skills`, other MoonPay repos, build instructions, and contribution guidelines.

## Coverage Map

Reading the list above in order covers:

- the overall OWS product story, design principles, and layered architecture
- vault storage format, wallet/API-key file structures, and cryptographic parameters
- key isolation, memory hardening, and the security threat model
- the four signing operations, their type definitions, and error handling
- the policy engine: declarative rules, custom policies, and default-deny semantics
- agent access model, credential types, and three deployment profiles
- wallet lifecycle from creation through deletion, including backup and recovery
- nine supported chain families, address derivation, and CAIP identifiers
- CLI, Node.js SDK, and Python SDK usage and API reference
- MoonPay Agents, skills framework, MCP integration, and the agent economy
- x402 payment protocol and the Bazaar directory for agent commerce
- public GitHub repositories, project structure, and how to contribute

## Key Facts

| Attribute | Value |
|-----------|-------|
| Version | v1.0.0 |
| Launch Date | March 23, 2026 |
| License | MIT |
| Created By | MoonPay |
| GitHub | [open-wallet-standard/core](https://github.com/open-wallet-standard/core) |
| Website | [openwallet.sh](https://openwallet.sh) |
| Docs | [docs.openwallet.sh](https://docs.openwallet.sh) |
| npm | `@open-wallet-standard/core` |
| PyPI | `open-wallet-standard` |
| crates.io | `ows-cli` |
| Primary Language | Rust (86.1%) |
| Supported Chains | 9 families (EVM, Solana, Bitcoin, Cosmos, Tron, TON, Sui, Spark, Filecoin) |