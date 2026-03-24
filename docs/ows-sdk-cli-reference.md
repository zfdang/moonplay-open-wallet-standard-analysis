# OWS SDK and CLI Reference

Grounded summary of the public reference implementation surfaces: the `ows` CLI, the Node.js package `@open-wallet-standard/core`, and the Python package `open-wallet-standard`.

## Normative vs Non-Normative

The numbered specification documents define interoperability. The SDK and CLI documents are **reference implementation documentation**, not the OWS standard itself.

The public repo states:

- `quickstart.md`, `sdk-cli.md`, `sdk-node.md`, and `sdk-python.md` are **non-normative reference implementation documentation**
- if there is a conflict between a normative core document and a reference implementation document, the normative core document wins

The public README also advertises a `Policy Engine Implementation Guide`, but that guide was not directly accessible during this review. This file therefore does not rely on it for concrete API claims.

That distinction matters when describing package names, installer behavior, CLI ergonomics, and SDK function signatures.

## Distribution Channels

| Surface | Package / Artifact | Notes |
|---|---|---|
| CLI | `ows` | Installed by the one-line installer, or by globally installing `@open-wallet-standard/core` |
| Node.js SDK | `@open-wallet-standard/core` | Native Node.js bindings via NAPI; Rust core runs in-process |
| Python SDK | `open-wallet-standard` | Native Python bindings via PyO3; Rust core runs in-process |
| Rust implementation | `ows/` workspace in the public repo | Build-from-source instructions are public, but this repository avoids overclaiming crate-level structure without a directly accessible source file |

## Installation

### One-line installer

```bash
curl -fsSL https://docs.openwallet.sh/install.sh | bash
```

The public quickstart and CLI reference describe this as the main install path for the current reference implementation.

### Install only what you need

```bash
npm install @open-wallet-standard/core
npm install -g @open-wallet-standard/core
pip install open-wallet-standard
```

The public repo README also documents building from source:

```bash
git clone https://github.com/open-wallet-standard/core.git
cd core/ows
cargo build --workspace --release
```

## Vault Path and File Layout

The default vault root for the reference implementation is `~/.ows/`, not `~/.ows/vault/`.

Public CLI docs show this layout:

```text
~/.ows/
  bin/
    ows
  wallets/
    <uuid>.json
  policies/
    <id>.json
  keys/
    <uuid>.json
  logs/
    audit.jsonl
```

The normative storage-format spec uses the same `~/.ows/` root and defines `config.json`, `wallets/`, `keys/`, `policies/`, and `logs/` under it.

One implementation-layer nuance matters here: the current CLI, Node, and Python examples for auto-derived accounts show **8 chain accounts**, while the normative supported-chains doc lists **9 families** including Spark. This repository treats the chain spec as the normative superset and the SDK docs as the current example auto-derivation set.

## CLI Reference

The public CLI docs explicitly say CLI syntax is **non-normative**. The following commands are the public documented command surface.

### Wallet Commands

```bash
# Create
ows wallet create --name "my-wallet"
ows wallet create --name "my-wallet" --words 24 --show-mnemonic

# Import from mnemonic (reads from OWS_MNEMONIC or stdin)
echo "goose puzzle decorate ..." | ows wallet import --name "imported" --mnemonic

# Import from private key (reads from OWS_PRIVATE_KEY or stdin)
echo "4c0883a691..." | ows wallet import --name "from-evm" --private-key
echo "9d61b19d..." | ows wallet import --name "from-sol" --private-key --chain solana

# Import explicit keys for both curves
OWS_SECP256K1_KEY="4c0883a691..." \
OWS_ED25519_KEY="9d61b19d..." \
  ows wallet import --name "both"

# Export / list / info / delete / rename
ows wallet export --wallet "my-wallet"
ows wallet list
ows wallet info
ows wallet delete --wallet "my-wallet" --confirm
ows wallet rename --wallet "old-name" --new-name "new-name"
```

Key documented flags include:

- `--name <NAME>` for creation/import
- `--words <12|24>` and `--show-mnemonic` for creation
- `--mnemonic`, `--private-key`, `--chain`, and `--index` for import

The public CLI docs also state that private-key imports generate accounts across the supported curve families, using the supplied key for its curve and generating a random key for the other curve unless both are supplied explicitly.

### Signing Commands

```bash
ows sign message --wallet "my-wallet" --chain evm --message "hello world"
ows sign tx --wallet "my-wallet" --chain evm --tx "02f8..."
```

The public CLI reference documents `ows sign message` and `ows sign tx`, not `ows sign-message`, `ows sign-typed-data`, or a flat `ows sign` command.

The signing docs also note:

- passphrases and API tokens are supplied via `OWS_PASSPHRASE` or an interactive prompt
- there is no dedicated `--passphrase` flag in the documented CLI surface
- `--typed-data <JSON>` exists on `ows sign message` for EVM typed-data signing

### Policy Commands

```bash
ows policy create --file policy.json
ows policy list
ows policy show --id "policy-id"
ows policy delete --id "policy-id" --confirm
```

### API Key Commands

```bash
ows key create --name "my-agent" --wallet my-wallet --policy agent-limits
ows key list
ows key revoke --id "key-id"
```

The public docs use `key`, not `apikey`.

### Mnemonic Commands

```bash
ows mnemonic generate
ows mnemonic generate --words 24
echo "goose puzzle decorate ..." | ows mnemonic derive --chain evm
```

The public CLI doc says mnemonic derivation reads the mnemonic from `OWS_MNEMONIC` or stdin.

### Payment Commands

```bash
ows pay request https://api.example.com/data --wallet "my-wallet"
ows pay request https://api.example.com/data --wallet "my-wallet" --method POST --body '{"key":"value"}'
ows pay discover
ows pay discover --query "weather"
ows pay discover --limit 20 --offset 100
```

The public OWS skill and CLI docs do mention a Bazaar directory for discovery, but that is part of the **reference implementation payment surface**, not part of the numbered core specification.

### Funding Commands

```bash
ows fund deposit --wallet "my-wallet"
ows fund deposit --wallet "my-wallet" --chain base
ows fund balance --wallet "my-wallet" --chain base
```

These commands are MoonPay-linked reference implementation features. They are not part of the normative core wallet format/signing/policy spec.

### System Commands

```bash
ows update
ows update --force
ows uninstall
ows uninstall --purge
```

## Node.js SDK

The public Node.js docs describe the package as native NAPI bindings where the Rust core runs in-process. The documented surface is **function-based**, not a client class.

### Install

```bash
npm install @open-wallet-standard/core
```

### Quick Start

```javascript
import {
  generateMnemonic,
  createWallet,
  listWallets,
  signMessage,
  signTypedData,
  deleteWallet,
} from "@open-wallet-standard/core";

const mnemonic = generateMnemonic(12);
const wallet = createWallet("my-wallet");
const sig = signMessage("my-wallet", "evm", "hello");
console.log(sig.signature);
```

The public Node SDK doc also says the package ships with prebuilt native binaries for macOS and Linux, so a Rust toolchain is not required for common installations.

### Publicly Documented Function Families

Mnemonic utilities:

```javascript
generateMnemonic(words?)
deriveAddress(mnemonic, chain, index?)
```

Wallet management:

```javascript
createWallet(name, passphrase?, words?, vaultPath?)
listWallets(vaultPath?)
getWallet(nameOrId, vaultPath?)
deleteWallet(nameOrId, vaultPath?)
renameWallet(nameOrId, newName, vaultPath?)
exportWallet(nameOrId, passphrase?, vaultPath?)
```

Import:

```javascript
importWalletMnemonic(name, mnemonic, passphrase?, index?, vaultPath?)
importWalletPrivateKey(name, privateKeyHex, passphrase?, vaultPath?, chain?, secp256k1Key?, ed25519Key?)
```

Signing:

```javascript
signMessage(wallet, chain, message, passphrase?, encoding?, index?, vaultPath?)
signTypedData(wallet, chain, typedDataJson, passphrase?, index?, vaultPath?)
signTransaction(wallet, chain, txHex, passphrase?, index?, vaultPath?)
signAndSend(wallet, chain, txHex, passphrase?, index?, rpcUrl?, vaultPath?)
```

Policy and API key management:

```javascript
createPolicy(jsonStr, vaultPath?)
listPolicies(vaultPath?)
deletePolicy(id, vaultPath?)

createApiKey(name, walletIds, policyIds, passphrase, expiresAt?, vaultPath?)
listApiKeys(vaultPath?)
revokeApiKey(id, vaultPath?)
```

### Return Types

The public binding source and SDK docs show:

- `createWallet(...)` returns `WalletInfo`
- signing functions return objects like `{ signature, recovery_id }` or `{ tx_hash }`
- API-key creation returns an object like `{ token, id, name }`

The Node SDK doc also adds two important implementation details:

- current typed-data signing is documented for **owner-mode credentials**; API-token typed-data signing is not yet documented as supported
- `signAndSend(...)` resolves `rpcUrl` from explicit input first, then from config or built-in defaults

That is materially different from a client-style API that returns mnemonics or nested objects.

## Python SDK

The public Python docs describe native bindings via **PyO3**, not CFFI.

### Install

```bash
pip install open-wallet-standard
```

### Quick Start

The public SDK docs currently show examples importing from `ows`:

```python
from ows import (
    generate_mnemonic,
    create_wallet,
    list_wallets,
    sign_message,
    sign_typed_data,
    delete_wallet,
)

mnemonic = generate_mnemonic(12)
wallet = create_wallet("my-wallet")
sig = sign_message("my-wallet", "evm", "hello")
print(sig["signature"])
```

The public quickstart also contains examples that import from `open_wallet_standard`. The important point for analysis is that the public Python API is **function-based**, mirroring the Node surface.

This is one of the public drift points called out elsewhere in the repo:

- `sdk-python.md` uses `from ows import ...`
- `quickstart.md` uses `from open_wallet_standard import ...`

### Publicly Documented Function Families

```python
generate_mnemonic(words=12)
derive_address(mnemonic, chain, index=None)

create_wallet(name, passphrase=None, words=None, vault_path=None)
import_wallet_mnemonic(name, mnemonic, passphrase=None, index=None, vault_path=None)
import_wallet_private_key(name, private_key_hex, chain=None, passphrase=None, vault_path=None, secp256k1_key=None, ed25519_key=None)

list_wallets(vault_path=None)
get_wallet(name_or_id, vault_path=None)
delete_wallet(name_or_id, vault_path=None)
rename_wallet(name_or_id, new_name, vault_path=None)
export_wallet(name_or_id, passphrase=None, vault_path=None)

sign_transaction(wallet, chain, tx_hex, passphrase=None, index=None, vault_path=None)
sign_message(wallet, chain, message, passphrase=None, encoding=None, index=None, vault_path=None)
sign_typed_data(wallet, chain, typed_data_json, passphrase=None, index=None, vault_path=None)
sign_and_send(wallet, chain, tx_hex, passphrase=None, index=None, rpc_url=None, vault_path=None)

create_policy(json_str, vault_path=None)
list_policies(vault_path=None)
get_policy(id, vault_path=None)
delete_policy(id, vault_path=None)

create_api_key(name, wallet_ids, policy_ids, passphrase, expires_at=None, vault_path=None)
list_api_keys(vault_path=None)
revoke_api_key(id, vault_path=None)
```

### Custom Vault Path

The public Node and Python docs both say every function accepts an optional vault-path parameter, and that the default vault root is `~/.ows/`.

Python-specific implementation details the public SDK page adds:

- prebuilt wheels are available for macOS and Linux on Python 3.9-3.13
- all return values are Python dicts rather than custom classes
- current typed-data signing is documented for owner-mode credentials only

## What This Repository Should Treat as Stable

For analysis purposes, the safest split is:

- **stable normative surface**: storage format, signing semantics, policy behavior, lifecycle semantics, supported chains, conformance/security requirements
- **reference implementation surface**: installer, CLI command names, SDK function signatures, funding helpers, payment-discovery UX, package/import details

That keeps the analysis aligned with the official OWS distinction between the standard and its current public implementation.

## Primary Sources

- `https://github.com/open-wallet-standard/core/blob/main/docs/sdk-cli.md`
- `https://raw.githubusercontent.com/open-wallet-standard/core/main/docs/sdk-node.md`
- `https://github.com/open-wallet-standard/core/blob/main/docs/sdk-python.md`
- `https://github.com/open-wallet-standard/core/blob/main/docs/quickstart.md`
- `https://github.com/open-wallet-standard/core/blob/main/docs/07-supported-chains.md`
