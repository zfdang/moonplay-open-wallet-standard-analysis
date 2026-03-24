# OWS SDK and CLI Reference

Practical guide to the three distribution channels — CLI, Node.js SDK, and Python SDK — with installation, commands, and API surface.

## Distribution Channels

| Channel | Package | Language | Primary Use |
|---------|---------|----------|-------------|
| CLI | `@open-wallet-standard/core` (global install) or `ows-cli` (crates.io) | Rust binary | Terminal workflows, scripting, CI/CD |
| Node.js SDK | `@open-wallet-standard/core` | TypeScript (NAPI binding to Rust) | Server-side agents, MCP tools |
| Python SDK | `open-wallet-standard` | Python (CFFI binding to Rust) | Research agents, Jupyter notebooks |

All three channels share the same Rust core. The SDKs are thin wrappers that call into the native library via NAPI (Node) or CFFI (Python).

## Installation

### CLI

```bash
# via npm (cross-platform)
npm install -g @open-wallet-standard/core

# verify
ows --version
```

```bash
# via cargo (requires Rust toolchain)
cargo install ows-cli

# via Homebrew (macOS — if tap is published)
brew install open-wallet-standard/tap/ows
```

### Node.js SDK

```bash
npm install @open-wallet-standard/core
# or
yarn add @open-wallet-standard/core
# or
pnpm add @open-wallet-standard/core
```

Requires Node.js ≥ 18. The package includes prebuilt native binaries for:
- macOS (arm64, x86_64)
- Linux (x86_64, arm64)
- Windows (x86_64)

### Python SDK

```bash
pip install open-wallet-standard
# or
uv pip install open-wallet-standard
```

Requires Python ≥ 3.9.

## CLI Commands

### Wallet Management

```bash
# Create a new wallet (generates a fresh 12-word mnemonic)
ows wallet create

# Create with 24 words
ows wallet create --words 24

# List all wallets in the vault
ows wallet list

# Import from mnemonic
ows wallet import --mnemonic "word1 word2 ... word12"

# Import from Keystore v3 JSON file
ows wallet import --keystore ./keystore.json

# Export mnemonic (requires passphrase)
ows wallet export --format mnemonic --wallet <wallet-id>

# Export as Keystore v3
ows wallet export --format keystore --wallet <wallet-id>

# Delete a wallet
ows wallet delete --wallet <wallet-id>
```

### Address Derivation

```bash
# Derive addresses for all supported chains (account index 0)
ows address derive --wallet <wallet-id>

# Derive for a specific chain
ows address derive --wallet <wallet-id> --chain eip155:8453

# Derive for a specific account index
ows address derive --wallet <wallet-id> --chain solana --index 3
```

### API Key Management

```bash
# Create an API key for a wallet
ows apikey create --wallet <wallet-id>

# Create with policies
ows apikey create --wallet <wallet-id> \
  --allowed-chains "eip155:8453,solana" \
  --expires "2026-12-31T23:59:59Z" \
  --policy ./my-policy.py

# List API keys
ows apikey list --wallet <wallet-id>

# Revoke an API key
ows apikey revoke --wallet <wallet-id> --key <key-id>

# Inspect an API key's metadata and policies
ows apikey inspect --wallet <wallet-id> --key <key-id>
```

### Signing

```bash
# Sign a transaction (owner mode — prompts for passphrase)
ows sign --wallet <wallet-id> --chain eip155:8453 --tx <hex-encoded-tx>

# Sign with an API key (agent mode)
ows sign --wallet <wallet-id> --chain eip155:8453 --tx <hex-encoded-tx> \
  --api-key <api-key-token>

# Sign and broadcast
ows sign --wallet <wallet-id> --chain eip155:8453 --tx <hex-encoded-tx> \
  --send --rpc https://mainnet.base.org

# Sign a message
ows sign-message --wallet <wallet-id> --chain eip155:1 --message "Hello, world!"

# Sign EIP-712 typed data
ows sign-typed-data --wallet <wallet-id> --chain eip155:1 --data ./typed-data.json
```

### Vault Operations

```bash
# Show vault status
ows vault status

# Backup the vault
ows vault backup --output ./vault-backup.tar.gz.enc

# Restore from backup
ows vault restore --input ./vault-backup.tar.gz.enc
```

### x402 Payments

```bash
# Make a payment via x402 protocol
ows pay request --to <recipient> --amount <amount> --asset <asset-id> \
  --chain <chain-id>

# Discover available payment endpoints
ows pay discover
```

## Node.js SDK API

```typescript
import { OWSClient } from '@open-wallet-standard/core';

// Initialize the client (uses default vault path ~/.ows/vault/)
const client = new OWSClient();

// --- Wallet Management ---

// Create a new wallet
const wallet = await client.createWallet({
  passphrase: 'my-secure-passphrase',
  words: 12  // optional, default 12
});
// Returns: { walletId: string, mnemonic: string, addresses: Record<ChainId, string> }

// List wallets
const wallets = await client.listWallets();
// Returns: Array<{ walletId: string, createdAt: string, chains: ChainId[] }>

// Import wallet
const imported = await client.importWallet({
  passphrase: 'my-secure-passphrase',
  mnemonic: 'word1 word2 ... word12'
});

// Delete wallet
await client.deleteWallet({ walletId: '...', passphrase: '...' });

// --- Address Derivation ---

const address = await client.deriveAddress({
  walletId: '...',
  chain: 'eip155:8453',
  index: 0
});
// Returns: { address: string, chain: ChainId, path: string }

// --- API Key Management ---

const apiKey = await client.createApiKey({
  walletId: '...',
  passphrase: '...',
  policies: {
    allowedChains: ['eip155:8453', 'solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp'],
    expiresAt: '2026-12-31T23:59:59Z'
  }
});
// Returns: { keyId: string, token: string }

await client.revokeApiKey({ walletId: '...', keyId: '...' });

// --- Signing ---

// Sign a transaction (agent mode)
const result = await client.sign({
  walletId: '...',
  chain: 'eip155:8453',
  transaction: '0x...',  // hex-encoded serialized transaction
  apiKey: '<api-key-token>'
});
// Returns: { signature: string }

// Sign and send
const txResult = await client.signAndSend({
  walletId: '...',
  chain: 'eip155:8453',
  transaction: '0x...',
  apiKey: '<api-key-token>',
  rpcUrl: 'https://mainnet.base.org'
});
// Returns: { txHash: string }

// Sign a message
const msgSig = await client.signMessage({
  walletId: '...',
  chain: 'eip155:1',
  message: 'Hello, world!',
  apiKey: '<api-key-token>'
});
// Returns: { signature: string }
```

## Python SDK API

```python
from open_wallet_standard import OWSClient

client = OWSClient()

# Create a wallet
wallet = client.create_wallet(passphrase="my-secure-passphrase", words=12)
print(wallet.wallet_id, wallet.mnemonic)

# List wallets
wallets = client.list_wallets()

# Derive address
addr = client.derive_address(
    wallet_id="...",
    chain="eip155:8453",
    index=0
)

# Create an API key
api_key = client.create_api_key(
    wallet_id="...",
    passphrase="...",
    allowed_chains=["eip155:8453"],
    expires_at="2026-12-31T23:59:59Z"
)

# Sign a transaction
result = client.sign(
    wallet_id="...",
    chain="eip155:8453",
    transaction="0x...",
    api_key=api_key.token
)
print(result.signature)

# Sign and send
tx = client.sign_and_send(
    wallet_id="...",
    chain="eip155:8453",
    transaction="0x...",
    api_key=api_key.token,
    rpc_url="https://mainnet.base.org"
)
print(tx.tx_hash)
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `OWS_VAULT_PATH` | `~/.ows/vault` | Path to the vault directory |
| `OWS_LOG_LEVEL` | `info` | Logging verbosity (trace, debug, info, warn, error) |
| `OWS_PASSPHRASE` | — | Passphrase for non-interactive use (avoid in production) |
| `OWS_API_KEY` | — | API key token for agent-mode signing |

## MoonPay CLI

MoonPay provides its own CLI that wraps OWS and adds MoonPay-specific features:

```bash
# Install
npm install -g @moonpay/cli

# Login to MoonPay
moonpay login

# Create a wallet (delegates to OWS)
moonpay wallet create

# Run a skill
moonpay skill run swap --from ETH --to USDC --amount 0.1

# Start MCP server for AI agent integration
moonpay mcp start
```

The MoonPay CLI uses OWS under the hood for all wallet and signing operations. It adds:
- Authentication with MoonPay services
- Skill execution framework
- MCP (Model Context Protocol) server for Claude, Cursor, and other AI tools
- On/off-ramp integration (buy/sell crypto with fiat)
