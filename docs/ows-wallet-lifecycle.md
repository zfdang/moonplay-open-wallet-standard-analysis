# OWS Wallet Lifecycle

How wallets are created, imported, exported, backed up, recovered, deleted, and rotated.

This document focuses on the **normative lifecycle semantics** from `06-wallet-lifecycle.md`, while also calling out places where the current reference implementation exposes additional CLI behavior.

## Creation

### Create from New Mnemonic

The lifecycle spec defines wallet creation as:

1. Generate 128 or 256 bits of cryptographically secure randomness
2. Encode as a BIP-39 mnemonic
3. Derive the master seed via PBKDF2
4. Derive accounts for the requested chains using the chain derivation rules
5. Encrypt the mnemonic with the vault passphrase using scrypt + AES-256-GCM
6. Write the encrypted wallet file under `~/.ows/wallets/`
7. Wipe mnemonic, seed, and derived keys from memory
8. Return a wallet descriptor containing public information only

The spec text explicitly says the mnemonic is **not** returned as part of the core lifecycle result.

### Create from Existing Private Key

The public lifecycle doc also covers importing a raw private key as wallet creation for single-chain or explicitly provided key material:

```bash
echo "<private-key>" | ows wallet import --name "imported" --chain evm --format raw
```

That path encrypts the provided key material immediately and zeroes the input buffer after import.

### Reference Implementation CLI

The public CLI reference documents:

```bash
ows wallet create --name "agent-treasury"
ows wallet create --name "agent-treasury" --words 24 --show-mnemonic
```

That means the current implementation exposes a dangerous opt-in UX for displaying the mnemonic once, but that should be read as reference-implementation behavior rather than a change to the normative lifecycle model.

## Import

The lifecycle spec documents these import paths:

1. Ethereum Keystore v3
2. BIP-39 mnemonic
3. WIF
4. Solana keypair JSON
5. Sui keystore JSON

Examples from the lifecycle spec include:

```bash
ows wallet import --name "from-geth" --format keystore --file ~/keystore/UTC--...
ows wallet import --name "from-metamask" --format mnemonic --chain evm
echo "<wif-key>" | ows wallet import --name "btc-wallet" --format wif --chain bitcoin
ows wallet import --name "sol-wallet" --format solana-keypair --file ~/.config/solana/id.json
ows wallet import --name "sui-wallet" --format sui-keystore --file ~/.sui/sui_config/sui.keystore
```

The public CLI reference for the current implementation documents a different surface centered on:

```bash
echo "goose puzzle decorate ..." | ows wallet import --name "imported" --mnemonic
echo "4c0883a691..." | ows wallet import --name "from-evm" --private-key
echo "9d61b19d..." | ows wallet import --name "from-sol" --private-key --chain solana
```

For analysis, the right reading is:

- the **spec** defines which lifecycle behaviors and source formats matter
- the **reference implementation** chooses a specific CLI syntax for invoking a subset of those flows

## Export

The lifecycle spec defines three export targets:

1. mnemonic
2. Keystore v3
3. raw private key

Examples from the spec:

```bash
ows wallet export --id 3198bc9c-... --format mnemonic
ows wallet export --id 3198bc9c-... --format keystore --output ~/exported.json
ows wallet export --id 3198bc9c-... --format raw --account eip155:8453:0xab16...
```

The public CLI reference for the implementation documents a simpler `ows wallet export --wallet "my-wallet"` surface in the SDK/CLI reference docs. The lifecycle analysis should therefore treat exact export flags as implementation detail, while preserving the three normative export semantics above.

The current CLI reference also says wallet export requires an interactive terminal and returns either:

- the mnemonic phrase for mnemonic wallets
- JSON containing both curve keys for private-key wallets

## Backup

The lifecycle spec explicitly includes:

- full-vault backup
- restore from backup
- automated backup configuration

The spec example uses:

```bash
ows backup --output ~/ows-backup-2026-02-27.tar.gz.enc
ows restore --input ~/ows-backup-2026-02-27.tar.gz.enc
```

The documented backup flow is:

1. tar the vault directory
2. encrypt the tar with a backup passphrase
3. write the encrypted archive to the requested path

The spec also documents an optional `backup` block in `~/.ows/config.json` for scheduled backups, including fields such as `enabled`, `schedule`, `destination`, `retention`, and `passphrase_env`.

The corresponding restore flow explicitly allows the implementation to prompt for **merge** versus **overwrite** if the destination vault already exists.

## Recovery

The lifecycle spec defines mnemonic recovery as a discovery flow:

1. accept the mnemonic interactively
2. derive accounts incrementally using the chain derivation path
3. query RPC endpoints for balance or transaction history
4. stop after 20 consecutive empty addresses
5. create a new wallet file containing the discovered accounts

That is a normative behavioral description; the exact RPC configuration remains implementation-specific.

The public lifecycle doc also treats restore-from-backup and import-from-keystore as recovery paths, not separate standards.

## Deletion

The lifecycle spec documents deletion as:

1. verify the wallet exists
2. prompt for confirmation unless forced
3. securely overwrite the wallet file before unlinking
4. remove the wallet ID from any API-key scopes that reference it
5. log the deletion to the audit log

That goes beyond a simple file delete; it is part of the lifecycle contract described by the public spec.

## Key Rotation

The lifecycle spec treats rotation as asset migration to a new key, not re-encryption of the existing wallet.

Documented flow:

1. create or select the destination wallet
2. query balances on the old wallet
3. construct transfer transactions for assets
4. sign and send using the old wallet
5. verify receipt on the new wallet

This distinction matters because rotation changes the active control key, not just the envelope around stored secrets.

## Wallet Discovery

The lifecycle spec also documents a discovery mechanism for environments where multiple tools may create OWS wallets.

The public examples include:

```text
discoverWallets({ chainType, chainId, name, hasPolicy })
ows wallet list --chain evm --with-policy
ows wallet list --name "agent-*"
```

That topic is easy to miss, but it is part of the public lifecycle document and should remain covered in this repository.

## Lifecycle Interpretation

The wallet lifecycle is one of the places where OWS most clearly separates:

- **normative semantics**: what create/import/export/backup/recover/delete/rotate must mean
- **reference implementation UX**: exact command names, flags, prompts, and installer choices

That separation keeps the analysis accurate when the public implementation evolves without changing the standard itself.

## References

- `docs/06-wallet-lifecycle.md` in `open-wallet-standard/core`
- `docs/00-specification.md` for the normative vs non-normative split
- `docs/sdk-cli.md` for the current public CLI syntax
