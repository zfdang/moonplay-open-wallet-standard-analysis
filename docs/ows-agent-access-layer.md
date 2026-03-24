# OWS Agent Access Layer

How AI agents and automated systems interact with OWS wallets through a capability-based access model.

## Purpose

The Agent Access Layer defines a uniform interface between an OWS wallet and any external caller — whether a human user, an AI agent, a CLI tool, or a background script. It specifies what operations exist, how credentials are presented, and what access profiles are available.

The goal is to allow **delegated access** without exposing private keys. An agent never sees the mnemonic or raw key material; it holds an API key that grants scoped, policy-gated access to sign operations.

## Access Model

OWS distinguishes two roles:

| Role | Credential | Capabilities |
|------|-----------|--------------|
| **Owner** | Passphrase | Full control: create, import, export, delete, backup, recover wallets; manage API keys; modify policies |
| **Agent** | API key | Scoped signing: `sign`, `signAndSend`, `signMessage`, `signTypedData` — subject to policies attached to the API key |

The passphrase is the root credential. API keys are derived artifacts.

## Required Operations

The Access Layer exposes these abstract operations:

### Wallet Management (Owner only)

| Operation | Description |
|-----------|-------------|
| `wallet.create` | Create a new wallet from a fresh mnemonic |
| `wallet.import` | Import a wallet from an existing mnemonic, private key, or keystore file |
| `wallet.export` | Export wallet material (mnemonic, private key, or Keystore v3) |
| `wallet.delete` | Securely delete a wallet and all associated files |
| `wallet.list` | List all wallets in the vault |
| `wallet.backup` | Create a backup of the vault |
| `wallet.restore` | Restore the vault from a backup |

### Key Management (Owner only)

| Operation | Description |
|-----------|-------------|
| `apikey.create` | Create a new API key for a wallet, with optional policy attachment |
| `apikey.list` | List all API keys for a wallet |
| `apikey.revoke` | Revoke an API key, making it permanently unusable |
| `apikey.inspect` | Show metadata and policies attached to an API key |

### Signing (Owner or Agent)

| Operation | Description |
|-----------|-------------|
| `sign` | Sign a serialized transaction, return the signature |
| `signAndSend` | Sign a transaction and broadcast it on-chain |
| `signMessage` | Sign an arbitrary message (EIP-191, Solana off-chain, etc.) |
| `signTypedData` | Sign structured typed data (EIP-712) |

### Discovery

| Operation | Description |
|-----------|-------------|
| `chain.list` | List supported chains |
| `address.derive` | Derive an address for a given chain and account index |

## Credential Semantics

### Passphrase (Owner)

- Required for all wallet management and key management operations
- Required as a fallback for signing when no API key is provided
- Verified by deriving the scrypt key and attempting AES-256-GCM decryption of the wallet file
- Never stored; always provided per-request (or cached in memory for a configurable TTL)

### API Key (Agent)

- A 256-bit random token, presented as a hex string
- Created by the owner via `apikey.create`
- The token's SHA-256 hash is stored in the API key file alongside the HKDF-derived encryption key
- On each signing request, the agent presents the raw token; the access layer hashes it, looks up the matching API key file, derives the decryption key via HKDF, and decrypts the wallet's private key
- The agent never sees the mnemonic, passphrase, or raw private key
- Each API key can have zero or more policies attached that constrain its use

## Access Profiles

OWS defines three access profiles, representing different deployment topologies:

### Profile A: In-Process

```
┌──────────────────────────────┐
│         Host Process         │
│  ┌────────┐  ┌────────────┐ │
│  │ Agent  │──│ OWS Signer │ │
│  └────────┘  └────────────┘ │
│              ▲               │
│              │               │
│         ~/.ows/vault/        │
└──────────────────────────────┘
```

- Agent and signer run in the same process
- Communication: direct function calls via the Node.js or Python SDK
- Lowest latency, simplest deployment
- Key material is in the same address space (mitigated by mlock + zeroize)

### Profile B: Subprocess

```
┌──────────────┐     ┌──────────────┐
│    Agent     │────▶│  OWS Signer  │
│  (process 1) │stdin│  (process 2) │
│              │◀────│              │
│              │stdout               │
└──────────────┘     └──────────────┘
                           ▲
                           │
                      ~/.ows/vault/
```

- Agent spawns the signer as a child process
- Communication: JSON over stdin/stdout (same protocol as custom policies)
- Key material lives in a separate address space
- Signer process can apply OS-level hardening (seccomp, pledge, unveil)

### Profile C: Local Service

```
┌──────────────┐     ┌──────────────┐
│    Agent     │────▶│  OWS Signer  │
│  (any host)  │HTTP │  (localhost)  │
│              │◀────│              │
└──────────────┘     └──────────────┘
                           ▲
                           │
                      ~/.ows/vault/
```

- Signer runs as a persistent local daemon
- Communication: HTTP over localhost (or Unix domain socket)
- Supports multiple concurrent agents
- Can be managed by systemd, launchd, or similar

### Profile Comparison

| Aspect | A: In-Process | B: Subprocess | C: Local Service |
|--------|--------------|---------------|-----------------|
| Latency | ~1 ms | ~5 ms | ~10 ms |
| Isolation | Same address space | Separate process | Separate process + network boundary |
| Concurrency | Single-threaded | Per-request | Multi-client |
| Deployment | Simplest | Moderate | Most complex |
| Key exposure | In agent memory | Signer-only memory | Signer-only memory |

## Cross-Layer Consistency

The Access Layer sits between the Policy Engine and the Signing Interface:

```
Agent Request
    │
    ▼
┌─────────────────┐
│  Access Layer    │  ← credential verification (passphrase or API key)
│  (this spec)     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Policy Engine   │  ← evaluate all attached policies
│  (spec 03)       │
└────────┬────────┘
         │ (if all policies pass)
         ▼
┌─────────────────┐
│  Signing Layer   │  ← derive key, sign transaction, return signature
│  (spec 02)       │
└─────────────────┘
```

Every signing request follows this exact sequence:
1. **Access Layer**: Verify credential. For an API key: hash the token, locate the API key file, verify the hash matches.
2. **Policy Engine**: Load all policies attached to the API key. Evaluate declarative rules first, then custom policies. If any policy denies, reject immediately.
3. **Signing Layer**: Derive the private key (via HKDF from API key, or via scrypt from passphrase), sign the transaction, zeroize the key, return the result.

A request that fails at any layer is rejected with a specific error code (see the Signing Interface spec).

## Security Properties

| Property | Mechanism |
|----------|-----------|
| Agent never sees private keys | HKDF derivation inside signer; key zeroized after use |
| API key revocation is instant | Deleting the API key file makes the token permanently invalid |
| Policies cannot be bypassed | Policy evaluation happens before signing, in the same trust boundary |
| Audit trail | Every signing attempt (success or failure) is logged to the audit log |
| Credential cannot be replayed across wallets | Each API key is bound to exactly one wallet file |
