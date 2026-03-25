# OWS Storage Format

The storage format is the core of the OWS standard. It defines how wallets, API keys, and policies are encrypted and stored on the local filesystem. Everything else — signing, policy enforcement, language bindings — operates on these files.

OWS extends the Ethereum Keystore v3 format with multi-chain support, API keys, and stronger encryption defaults.

The numbered storage doc treats conformance here very concretely: an implementation conforms by correctly reading and writing these artifacts and by preserving the documented semantics around encryption, IDs, permissions, and compatibility.

## Vault Directory Structure

```
~/.ows/
├── config.json                    # Global configuration
├── wallets/
│   ├── <wallet-id>.json           # Encrypted wallet file (one per wallet)
│   └── ...
├── keys/
│   ├── <key-id>.json              # API key + encrypted wallet secrets (one per key)
│   └── ...
├── policies/
│   ├── <policy-id>.json           # Policy definition (declarative rules and/or executable)
│   └── ...
└── logs/
    └── audit.jsonl                # Append-only audit log
```

### Filesystem Permissions

```
~/.ows/                       drwx------  (700)
~/.ows/wallets/               drwx------  (700)
~/.ows/wallets/*.json         -rw-------  (600)
~/.ows/keys/                  drwx------  (700)
~/.ows/keys/*.json            -rw-------  (600)
~/.ows/policies/              drwxr-xr-x  (755)
~/.ows/policies/*.json        -rw-r--r--  (644)
~/.ows/config.json            -rw-------  (600)
~/.ows/logs/audit.jsonl       -rw-------  (600)
```

- `wallets/` and `keys/` contain encrypted secrets → readable only by the owner (700/600)
- `policies/` uses relaxed permissions (755/644) because policy files are not secret — they contain rule definitions and paths to executables, not key material
- Implementations MUST verify permissions on startup and refuse to operate if secret directories are world-readable or group-readable

## Policy Files In The Vault

Policy files live under `~/.ows/policies/<policy-id>.json`.

This storage document only needs a few format-level points:

- policy files are JSON artifacts stored alongside wallets, keys, and logs
- policy files are not treated as secret material, so their permissions are looser than `wallets/` and `keys/`
- API key files refer to policies by `policy_ids`

The **full policy file schema** belongs to the policy engine contract rather than the storage envelope itself. See [ows-policy-engine.md](ows-policy-engine.md) for:

- the policy JSON example
- field definitions such as `id`, `name`, `rules`, `executable`, `config`, and `action`
- evaluation semantics and default-deny behavior

## Wallet File Format

Each wallet is a single JSON file extending the Ethereum Keystore v3 structure.

Example: mnemonic-backed wallet

```json
{
  "ows_version": 2,
  "id": "3198bc9c-6672-5ab3-d995-4942343ae5b6",
  "name": "agent-treasury",
  "created_at": "2026-02-27T10:30:00Z",
  "accounts": [
    {
      "account_id": "eip155:8453:0xab16a96D359eC26a11e2C2b3d8f8B8942d5Bfcdb",
      "address": "0xab16a96D359eC26a11e2C2b3d8f8B8942d5Bfcdb",
      "chain_id": "eip155:8453",
      "derivation_path": "m/44'/60'/0'/0/0"
    }
  ],
  "crypto": {
    "cipher": "aes-256-gcm",
    "cipherparams": {
      "iv": "6087dab2f9fdbbfaddc31a90"
    },
    "ciphertext": "5318b4d5bcd28de64ee5559e671353e16f075ecae9f99c7a79a38af5f869aa46",
    "auth_tag": "3c5d8c2f1a4b6e9d0f2a5c8b",
    "kdf": "scrypt",
    "kdfparams": {
      "dklen": 32,
      "n": 65536,
      "r": 8,
      "p": 1,
      "salt": "ae3cd4e7013836a3df6bd7241b12db061dbe2c6785853cce422d148a624ce0bd"
    }
  },
  "key_type": "mnemonic",
  "metadata": {}
}
```

Example: imported private-key wallet

```json
{
  "ows_version": 2,
  "id": "8b53b62e-1327-49dc-bcdc-6a2f51ce27e1",
  "name": "base-hot-wallet",
  "created_at": "2026-03-01T08:15:00Z",
  "accounts": [
    {
      "account_id": "eip155:8453:0x742d35Cc6634C0532925a3b844Bc9e7595f2bD0C",
      "address": "0x742d35Cc6634C0532925a3b844Bc9e7595f2bD0C",
      "chain_id": "eip155:8453"
    }
  ],
  "crypto": {
    "cipher": "aes-256-gcm",
    "cipherparams": {
      "iv": "4f2bb8e31d9ab4ce6a770912"
    },
    "ciphertext": "78fb342bf1fc4f75d561f17a020d5cb138e9c4b9c70489e6ddbf8f12c8a75121",
    "auth_tag": "9a4e6f7b8c1d2e3f4a5b6c7d",
    "kdf": "scrypt",
    "kdfparams": {
      "dklen": 32,
      "n": 65536,
      "r": 8,
      "p": 1,
      "salt": "4ed27a6bf25dce0f2bd961b7c9146946e97da9b4f0542f8c517eddd33f49244c"
    }
  },
  "key_type": "private_key",
  "metadata": {}
}
```

### Wallet File Field Definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `ows_version` | integer | yes | Schema version (currently 2) |
| `id` | string | yes | UUID v4 wallet identifier |
| `name` | string | yes | Human-readable wallet name |
| `created_at` | string | yes | ISO 8601 creation timestamp |
| `accounts` | array | yes | Derived accounts (see Account object) |
| `crypto` | object | yes | Encryption parameters (see Crypto object) |
| `key_type` | string | yes | `mnemonic` (BIP-39) or `private_key` (raw) |
| `metadata` | object | no | Extensible metadata |

### What Gets Encrypted

The `ciphertext` contains the encrypted form of either:

- **BIP-39 mnemonic entropy** (128 or 256 bits) — when `key_type` is `mnemonic`. The mnemonic can derive keys for any supported chain via BIP-44 derivation paths.
- **Raw private key** (32 bytes for secp256k1/ed25519) — when `key_type` is `private_key`. Used for imported single-chain keys.

Storing the mnemonic (rather than individual private keys) enables a single encrypted blob to derive accounts across multiple chains and indices.

### How To Read The Wallet Encryption Flow

At a high level, a mnemonic-backed wallet JSON can be understood like this:

1. The user has a mnemonic.
2. The user enters a wallet passphrase.
3. OWS runs `scrypt(passphrase, salt, n, r, p, dklen)` using the values in `crypto.kdfparams` to derive a 32-byte key.
4. OWS uses that derived key with `aes-256-gcm` and the stored IV to encrypt the wallet secret.
5. OWS saves:
   - `ciphertext`
   - `cipherparams.iv`
   - `auth_tag`
   - the KDF parameters (`salt`, `n`, `r`, `p`, `dklen`)
   - public account metadata such as addresses and derivation paths

To unlock or recover the wallet later:

1. The user enters the same passphrase again.
2. OWS reruns `scrypt(...)` with the same stored parameters.
3. OWS uses the re-derived key and `aes-256-gcm` to decrypt `ciphertext`.
4. OWS recovers the wallet secret.
5. For mnemonic-backed wallets, OWS derives chain-specific accounts again from the mnemonic and the recorded derivation paths.

For intuition, it is fine to think of this as "encrypt the mnemonic with a key derived from the passphrase." More precisely, the encrypted payload is the wallet secret represented in the file: mnemonic entropy for `key_type: "mnemonic"` wallets, or a raw private key for `key_type: "private_key"` wallets.

## API Key File Format

Each API key is stored as a JSON file in `~/.ows/keys/`. The key file contains metadata, policy attachments, and encrypted copies of wallet secrets re-encrypted under the API token:

```json
{
  "id": "7a2f1b3c-4d5e-6f7a-8b9c-0d1e2f3a4b5c",
  "name": "claude-agent",
  "token_hash": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "created_at": "2026-03-22T10:30:00Z",
  "wallet_ids": ["3198bc9c-6672-5ab3-d995-4942343ae5b6"],
  "policy_ids": ["spending-limit", "base-only"],
  "expires_at": null,
  "wallet_secrets": {
    "3198bc9c-6672-5ab3-d995-4942343ae5b6": {
      "cipher": "aes-256-gcm",
      "cipherparams": { "iv": "a1b2c3d4e5f6a7b8c9d0e1f2" },
      "ciphertext": "...",
      "auth_tag": "...",
      "kdf": "hkdf-sha256",
      "kdfparams": { "dklen": 32, "salt": "...", "info": "ows-api-key-v1" }
    }
  }
}
```

### API Key Field Definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | yes | UUID v4 key identifier |
| `name` | string | yes | Human-readable label for the key |
| `token_hash` | string | yes | SHA-256 hex digest of the raw token. The raw token (`ows_key_<64 hex chars>`) is shown once at creation and never stored. |
| `created_at` | string | yes | ISO 8601 creation timestamp |
| `wallet_ids` | array | yes | Wallet IDs this key is authorized to access |
| `policy_ids` | array | yes | Policy IDs evaluated on every request made with this key |
| `expires_at` | string | no | ISO 8601 expiry timestamp. `null` means no expiry. |
| `wallet_secrets` | object | yes | Map of wallet ID → CryptoEnvelope. Each entry is that wallet's secret re-encrypted under HKDF(token). In the common case this is mnemonic entropy; for `private_key` wallets it is the raw private key. |

### What Gets Encrypted

In an API key file, each `wallet_secrets[wallet_id].ciphertext` contains an encrypted copy of the underlying wallet secret for that wallet:

- **Mnemonic-backed wallet**: the wallet's mnemonic entropy is re-encrypted for token-based access.
- **Private-key wallet**: the wallet's raw private key is re-encrypted for token-based access.

The important distinction from `~/.ows/wallets/*.json` is that the wallet file is encrypted under the owner's passphrase, while the API key file stores an additional encrypted copy under a key derived from the API token.

### Is There A Separate `key_type` Inside `wallet_secrets`?

In the current public API key example and field list, `wallet_secrets[wallet_id]` is just a `CryptoEnvelope`. It does **not** define a separate inner field like:

- `wallet_secrets[wallet_id].key_type`

So in the minimal public format, the type is typically inferred from the wallet identified by `wallet_id`:

- if the referenced wallet file has `key_type: "mnemonic"`, the re-encrypted secret is mnemonic entropy
- if the referenced wallet file has `key_type: "private_key"`, the re-encrypted secret is a raw private key

That is why the key file stores `wallet_ids` and `wallet_secrets` keyed by wallet ID rather than repeating the full wallet schema inside each entry.

The public spec does allow wallet and API key files to carry additional metadata fields, so an implementation could redundantly store extra type hints. But that is an extension, not part of the minimal public schema described here.

### Where These Encrypted Versions Live

If the same wallet is usable by both the owner and one or more agents, the encrypted versions are **not** all stored in the same file.

The layout is:

- **Owner copy**: one wallet file in `~/.ows/wallets/<wallet-id>.json`
- **Agent copies**: one API key file per API token in `~/.ows/keys/<key-id>.json`

That means a single wallet typically has:

- one passphrase-encrypted version in its wallet file
- zero or more token-encrypted versions in separate API key files

For example, if wallet `W1` is granted to two agents:

- `~/.ows/wallets/W1.json` stores the owner-encrypted wallet secret
- `~/.ows/keys/K1.json` may contain `wallet_secrets["W1"]`
- `~/.ows/keys/K2.json` may also contain `wallet_secrets["W1"]`

So the design is:

- **not** "many encrypted versions inside the wallet file"
- **not** "all agent copies in one shared global file"
- instead, **one wallet file plus one key file per agent/API token**

This is why revocation is simple: deleting or revoking one API key removes only that agent's encrypted copy, without changing the original wallet file or other agents' copies.

### How Deletion Works With Multiple Copies

There are two different deletion cases:

1. **Delete one agent's copy only**
2. **Delete the wallet entirely**

If you want to remove access for just one agent:

- revoke that API key
- delete its `~/.ows/keys/<key-id>.json`

That removes only that agent's token, scope, policies, and encrypted wallet-secret copies. The original wallet file and other agents' key files remain unchanged.

If you want to delete the wallet itself:

- delete the owner wallet file in `~/.ows/wallets/<wallet-id>.json`
- remove that `wallet_id` from any API key scopes that reference it

The public lifecycle document is explicit about removing the wallet ID from referencing API keys. Operationally, the corresponding `wallet_secrets[wallet_id]` entry should also be removed from those key files; otherwise an orphaned encrypted copy would remain. That storage implication is the natural consequence of the lifecycle rule, even though the public lifecycle text calls out the scope cleanup more directly than the inner map cleanup.

### How To Read The API Key Encryption Flow

At a high level, an API key JSON can be understood like this:

1. The owner already has an existing wallet file.
2. The owner unlocks that wallet with the wallet passphrase.
3. OWS decrypts the wallet secret from the wallet file.
4. OWS generates a random API token such as `ows_key_<64 hex chars>`.
5. OWS generates a fresh salt for the API key envelope.
6. OWS runs `HKDF-SHA256(token, salt, info="ows-api-key-v1", dklen=32)` to derive a 32-byte key.
7. OWS uses that derived key with `aes-256-gcm` and the stored IV to encrypt the wallet secret again.
8. OWS saves:
   - `token_hash`
   - `wallet_ids`
   - `policy_ids`
   - `expires_at`
   - `wallet_secrets[wallet_id].ciphertext`
   - `wallet_secrets[wallet_id].cipherparams.iv`
   - `wallet_secrets[wallet_id].auth_tag`
   - the HKDF parameters (`salt`, `info`, `dklen`)

To use that API key later:

1. The agent presents the same raw token.
2. OWS computes `SHA256(token)` and finds the matching API key file.
3. OWS checks scope, expiry, and attached policies before decryption.
4. OWS reruns `HKDF-SHA256(...)` with the stored parameters.
5. OWS uses the re-derived key and `aes-256-gcm` to decrypt `wallet_secrets[wallet_id].ciphertext`.
6. OWS recovers the wallet secret, derives the chain-specific key if needed, signs, and then wipes the decrypted material from memory.

For intuition, it is fine to think of this as "re-wrapping the wallet secret under the API token." The owner passphrase protects the original wallet file; the API token protects a second encrypted copy used for policy-constrained agent access.

### What An Agent Request Usually Reads

In the public OWS model, an agent request usually starts from a single API key file:

1. OWS hashes the presented token.
2. OWS finds the matching `~/.ows/keys/<key-id>.json`.
3. From that file, OWS gets:
   - the token match (`token_hash`)
   - the wallet scope (`wallet_ids`)
   - the policy references (`policy_ids`)
   - the encrypted wallet-secret copy (`wallet_secrets[wallet_id]`)

So for the **agent-specific secret material**, the key file is the main file that matters.

But a full request typically involves more than that one file:

- OWS still needs the referenced policy files from `~/.ows/policies/`
- implementations may also look up the wallet file or wallet index for public metadata, wallet resolution, or account information

So the safe mental model is:

- **decryption path**: mostly driven by `key.json`
- **full request handling**: usually `key.json` plus policy files, and sometimes wallet metadata

## Crypto Object

The `crypto` object follows Keystore v3 conventions with two upgrades:

1. **AES-256-GCM** is the default cipher (upgraded from AES-128-CTR). GCM provides authenticated encryption, eliminating the need for a separate MAC field.
2. **scrypt** remains the recommended KDF for wallet files (passphrase-derived). **HKDF-SHA256** is used for API key files (token-derived).

| Field | Type | Description |
|-------|------|-------------|
| `cipher` | string | `aes-256-gcm` (recommended) or `aes-128-ctr` (v3 compat) |
| `cipherparams.iv` | string | Hex-encoded initialization vector |
| `ciphertext` | string | Hex-encoded encrypted key material |
| `auth_tag` | string | Hex-encoded GCM auth tag (only for `aes-256-gcm`) |
| `mac` | string | Keystore-v3-style MAC field for `aes-128-ctr` compatibility envelopes |
| `kdf` | string | `scrypt`, `hkdf-sha256`, or `pbkdf2` |
| `kdfparams` | object | KDF-specific parameters |

### scrypt kdfparams (wallet files — passphrase input)

| Param | Type | Description |
|-------|------|-------------|
| `dklen` | integer | Derived key length in bytes (32) |
| `n` | integer | CPU/memory cost parameter (must be power of 2, minimum 2^16) |
| `r` | integer | Block size (8) |
| `p` | integer | Parallelization (1) |
| `salt` | string | Hex-encoded random salt (32 bytes) |

### hkdf-sha256 kdfparams (API key files — token input)

| Param | Type | Description |
|-------|------|-------------|
| `dklen` | integer | Derived key length in bytes (32) |
| `salt` | string | Hex-encoded random salt (32 bytes) |
| `info` | string | Context string (`"ows-api-key-v1"`) |

## Audit Log

All signing operations are appended to `~/.ows/logs/audit.jsonl`:

```json
{
  "timestamp": "2026-02-27T10:35:22Z",
  "wallet_id": "3198bc9c-6672-5ab3-d995-4942343ae5b6",
  "operation": "broadcast_transaction",
  "chain_id": "eip155:8453",
  "details": "tx_hash=0xabc123..."
}
```

Audited operations: `create_wallet`, `import_wallet`, `export_wallet`, `broadcast_transaction`, `delete_wallet`, `rename_wallet`.

The audit log is **append-only**. Implementations MUST NOT allow deletion or modification of existing entries. Log rotation is permitted (e.g., monthly archives).

The public conformance doc narrows the minimum useful fields further: audit records should include operation type, wallet identifier, chain identifier, timestamp, allow / deny outcome, and API key identifier when applicable, but must not include raw secrets.

## Passphrase Management

OWS does NOT define how the passphrase is obtained — this is deliberately left to the implementation:

| Mode | How Passphrase Is Provided |
|------|---------------------------|
| Interactive CLI | Prompt at first use, optionally cache in OS keychain for a session |
| Agent/daemon mode | Read from a file descriptor (RECOMMENDED), an environment variable (`OWS_PASSPHRASE`), or a hardware token |
| Unlocked mode (dev only) | A config flag that uses a well-known passphrase — MUST produce a visible warning |

The passphrase MUST be at least 12 characters. Implementations SHOULD enforce this at wallet creation time.

**Warning:** Environment variables are the least secure option — they are readable via `/proc/[pid]/environ` by same-user processes and leak into crash dumps and child process environments. Implementations using `OWS_PASSPHRASE` MUST clear it from the process environment immediately after reading.

## Backward Compatibility

- Any valid Ethereum Keystore v3 file can be imported into an OWS vault
- Exported OWS wallets with `cipher: "aes-128-ctr"` and `key_type: "private_key"` are valid Keystore v3 files (minus the OWS envelope fields, which are ignored by v3 parsers)

The public storage doc is explicit that compatibility is directional and procedural:

1. read v3 JSON
2. decrypt using the source wallet's rules
3. re-wrap in the OWS envelope
4. preserve OWS semantics after import

## References

- `https://github.com/open-wallet-standard/core/blob/main/docs/01-storage-format.md`
- `https://github.com/open-wallet-standard/core/blob/main/docs/08-conformance-and-security.md`
- [Ethereum Web3 Secret Storage Definition](https://ethereum.org/developers/docs/data-structures-and-encoding/web3-secret-storage)
- [ERC-2335: BLS12-381 Keystore](https://eips.ethereum.org/EIPS/eip-2335)
- [BIP-39: Mnemonic Seed Phrases](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)
- [NIST SP 800-38D: GCM Mode](https://csrc.nist.gov/publications/detail/sp/800-38d/final)
