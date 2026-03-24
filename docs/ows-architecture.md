# OWS Architecture

## Layered Architecture

```
OWS
├── Access Layer
│   ├── CLI (`ows` command)
│   ├── Node.js SDK (@open-wallet-standard/core)
│   ├── Python SDK (open-wallet-standard)
│   ├── MCP Server (Model Context Protocol)
│   └── REST Interface (optional local daemon)
├── Policy Engine
│   ├── Declarative Rules (allowed_chains, expires_at)
│   ├── Custom Executable Policies (stdin/stdout protocol)
│   └── AND semantics (all policies must pass)
├── Signing Core
│   ├── In-process Rust library
│   ├── Chain-specific signers (9 chain families)
│   ├── mlock'd memory + zeroization
│   └── Key caching (short-lived, bounded)
├── Wallet Vault
│   ├── ~/.ows/wallets/ (AES-256-GCM + scrypt encrypted)
│   ├── ~/.ows/keys/ (API key files, HKDF-encrypted)
│   ├── ~/.ows/policies/ (JSON rule definitions)
│   └── ~/.ows/logs/audit.jsonl (append-only)
└── Supported Chains
    ├── EVM (Ethereum, Polygon, Arbitrum, Optimism, Base, BSC, Avalanche)
    ├── Solana
    ├── Bitcoin (BIP-84 native segwit)
    ├── Cosmos
    ├── Tron
    ├── TON
    ├── Sui
    ├── Spark (Bitcoin L2)
    └── Filecoin
```

## Architecture Diagram

```
Agent / CLI / App
       │
       │  OWS Interface (SDK / CLI / MCP)
       ▼
┌─────────────────────────┐
│      Access Layer        │     1. Caller invokes sign()
│  ┌───────────────────┐   │     2. Credential detected (passphrase or API token)
│  │   Policy Engine    │   │     3. If API token: policies evaluated BEFORE decryption
│  │  (pre-signing)     │   │     4. Key decrypted in hardened memory (mlock)
│  └────────┬──────────┘   │     5. Chain-specific HD derivation
│  ┌────────▼──────────┐   │     6. Transaction signed
│  │  Signing Core      │   │     7. Key immediately wiped (zeroize)
│  │  (in-process,      │   │     8. Signature returned
│  │   Rust via FFI)    │   │
│  └────────┬──────────┘   │     The OWS API never returns
│  ┌────────▼──────────┐   │     raw private keys.
│  │  Wallet Vault      │   │
│  │  ~/.ows/wallets/   │   │
│  └───────────────────┘   │
└─────────────────────────┘
```

## Signing Flow — Agent Mode

```mermaid
sequenceDiagram
    participant Agent
    participant OWS as OWS Library
    participant Policy as Policy Engine
    participant Vault as Wallet Vault
    participant Signer as Chain Signer

    Agent->>OWS: sign_transaction(wallet, chain, tx, "ows_key_...")
    OWS->>OWS: Detect ows_key_ prefix → agent mode
    OWS->>OWS: SHA256(token) → lookup API key file
    OWS->>OWS: Check expires_at, verify wallet in scope
    OWS->>Policy: Evaluate all attached policies
    Policy-->>OWS: ALLOW (all pass) or DENY (short-circuit)
    
    alt Policy Denied
        OWS-->>Agent: POLICY_DENIED error (key material never touched)
    else Policy Allowed
        OWS->>Vault: HKDF-SHA256(salt, token) → decrypt mnemonic
        Vault-->>OWS: Encrypted mnemonic → hardened memory (mlock)
        OWS->>Signer: HD-derive chain-specific key + sign
        Signer-->>OWS: Signature
        OWS->>OWS: Zeroize mnemonic + derived key + KDF key
        OWS-->>Agent: Return signature only
    end
```

## Signing Flow — Owner Mode

```mermaid
sequenceDiagram
    participant Owner
    participant OWS as OWS Library
    participant Vault as Wallet Vault
    participant Signer as Chain Signer

    Owner->>OWS: sign_transaction(wallet, chain, tx, passphrase)
    OWS->>OWS: Detect passphrase (not ows_key_ prefix) → owner mode
    Note over OWS: NO policy evaluation — owner has sudo access
    OWS->>Vault: scrypt(passphrase) → decrypt mnemonic
    Vault-->>OWS: Mnemonic → hardened memory (mlock)
    OWS->>Signer: HD-derive chain-specific key + sign
    Signer-->>OWS: Signature
    OWS->>OWS: Zeroize all key material
    OWS-->>Agent: Return signature only
```

## Access Model

| Tier | Credential | Policy Enforcement |
|------|-----------|-------------------|
| Owner | Wallet passphrase | None. Full access to all wallets. Sudo mode. |
| Agent | `ows_key_...` token | All policies attached to the API key are evaluated. Every policy must allow (AND semantics). |

The credential itself determines the access tier. No bypass flags. No configuration to toggle. The owner uses the passphrase; agents use tokens. Different agents get different tokens with different policies.

If the owner wants policy-constrained access for themselves, they create an API key and use the token instead of the passphrase.

## Access Profiles

The spec defines three conforming access profiles:

### Profile A: In-Process Binding

The caller links directly against the OWS Rust library via FFI (Node.js NAPI, Python CFFI). Lowest latency, same address space.

### Profile B: Local Subprocess

The caller spawns an OWS child process per operation. Better isolation.

### Profile C: Local Service

A loopback-only daemon or local RPC endpoint. Must bind only to local interfaces.

All profiles MUST preserve the same signing semantics, policy evaluation order, error codes, and audit log behavior.

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Core implementation | Rust |
| Encryption | AES-256-GCM (upgraded from Keystore v3's AES-128-CTR) |
| KDF (wallets) | scrypt (passphrase → encryption key) |
| KDF (API keys) | HKDF-SHA256 (token → encryption key) |
| Memory hardening | mlock, zeroize, anti-ptrace, anti-coredump |
| Chain identifiers | CAIP-2 / CAIP-10 |
| Key derivation | BIP-32 / BIP-39 / BIP-44 |
| Node.js binding | NAPI (native FFI) |
| Python binding | CFFI (native FFI) |
| License | MIT |
