# OWS Policy Engine

How transaction policies are defined, evaluated, and enforced before any key material is touched.

The policy engine is the primary security boundary between AI agents and key material. It ensures that agents can only perform operations that the wallet owner has explicitly authorized.

## Access Model

```
sign_transaction(wallet, chain, tx, credential)
                                       │
                          ┌────────────┴────────────┐
                          │                          │
                     passphrase                 ows_key_...
                          │                          │
                     owner mode                 agent mode
                     no policy                  policies enforced
                     scrypt decrypt             HKDF decrypt
```

| Tier | Credential | Policy Enforcement |
|------|-----------|-------------------|
| Owner | Wallet passphrase | None. Full access to all wallets. |
| Agent | `ows_key_...` token | All policies attached to the API key are evaluated. Every policy must allow (AND semantics). |

The credential itself determines the access tier. The owner uses the passphrase; agents use tokens. Different agents get different tokens with different policies.

## API Key Cryptography

### Token-as-Capability Model

When the owner creates an API key, OWS:

1. Decrypts the wallet mnemonic using the owner's passphrase
2. Re-encrypts the mnemonic under a key derived from the API token
3. Stores the encrypted copy in the API key file

The agent presents the token with each signing request; the token serves as **both authentication and decryption capability**.

### Key Derivation (HKDF-SHA256)

API tokens are 256-bit random values (`ows_key_<64 hex chars>`). HKDF-SHA256 derives the encryption key:

```
token = ows_key_<random 256 bits, hex-encoded>
salt  = random 32 bytes (stored in CryptoEnvelope)
prk   = HKDF-Extract(salt, token)
key   = HKDF-Expand(prk, "ows-api-key-v1", 32)  →  AES-256-GCM key
```

### Key Creation Flow

```
ows key create --name "claude-agent" --wallet agent-treasury --policy spending-limit
```

1. Owner enters wallet passphrase
2. OWS decrypts the wallet mnemonic using scrypt(passphrase)
3. Generates random token: `T = "ows_key_" + hex(random 256 bits)`
4. Generates random salt S
5. Derives key: `K = HKDF-SHA256(S, T, "ows-api-key-v1", 32)`
6. Encrypts mnemonic with K via AES-256-GCM
7. Stores key file with `token_hash: SHA256(T)`, policy IDs, and encrypted mnemonic copy
8. **Displays T once** — owner provisions it to the agent
9. Zeroizes mnemonic from memory

### Agent Signing Flow

```
Agent calls: sign_transaction(wallet, chain, tx, "ows_key_a1b2c3...")

1. Detect ows_key_ prefix → agent mode
2. SHA256(token) → look up API key file
3. Check expires_at (if set)
4. Verify wallet is in key's wallet_ids scope
5. Load policies from key's policy_ids
6. Build PolicyContext (chain ID, wallet ID, API key ID, transaction context, spending context, timestamp)
7. Evaluate all policies (AND semantics, short-circuit on first deny)
8. If denied → return POLICY_DENIED error (key material never touched)
9. HKDF-SHA256(salt, token) → AES key → decrypt mnemonic from key.wallet_secrets
10. HD-derive chain-specific key
11. Sign transaction
12. Zeroize mnemonic and derived key
13. Return signature
```

### Revocation

Delete the API key file. The encrypted mnemonic copy is gone. `SHA256(T)` matches nothing. The token is useless. The original wallet and other API keys are unaffected.

### Security Properties

| Scenario | Result |
|----------|--------|
| Token stolen, no disk access | Useless — encrypted key file not accessible |
| Disk access, no token | Can't decrypt — HKDF + AES-256-GCM |
| Token + disk access | Can decrypt, but requires bypassing OWS process entirely |
| Owner passphrase changed | API keys unaffected (independently encrypted) |
| API key revoked | Encrypted copy deleted — token decrypts nothing |
| Multiple API keys | Independent encrypted copies; revoking one doesn't affect others |

## Declarative Policy Rules

These rule types are evaluated in-process (microseconds, no subprocess):

### `allowed_chains`

Restricts which CAIP-2 chain IDs can be signed for.

```json
{ "type": "allowed_chains", "chain_ids": ["eip155:8453", "eip155:84532"] }
```

### `expires_at`

Time-bound access (compares `PolicyContext.timestamp` to this ISO-8601 string).

```json
{ "type": "expires_at", "timestamp": "2026-04-01T00:00:00Z" }
```

## Custom Executable Policies

For anything declarative rules can't express — on-chain simulation, external API calls, complex business logic.

### Protocol

```
echo '<PolicyContext JSON>' | /path/to/policy-executable
```

- The executable receives the full `PolicyContext` as a single JSON object on stdin
- The executable MUST write a single `PolicyResult` JSON object to stdout
- A non-zero exit code is treated as a denial
- Stderr is captured and may be surfaced in denial details

### Evaluation Order Within a Policy

A policy can have both `rules` (declarative) and `executable` (custom). When both are present:

1. Declarative rules evaluate first (in-process, fast)
2. If declarative rules deny → skip executable (no subprocess spawned)
3. If declarative rules allow → spawn executable for final verdict
4. **Both must allow**

Declarative rules act as a fast pre-filter.

## Policy File Format

Policies are JSON files stored in `~/.ows/policies/`:

```json
{
  "id": "base-agent-limits",
  "name": "Base Agent Safety Limits",
  "version": 1,
  "created_at": "2026-03-22T10:00:00Z",
  "rules": [
    { "type": "allowed_chains", "chain_ids": ["eip155:8453", "eip155:84532"] },
    { "type": "expires_at", "timestamp": "2026-12-31T23:59:59Z" }
  ],
  "executable": null,
  "config": null,
  "action": "deny"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | yes | Unique policy identifier |
| `name` | string | yes | Human-readable policy name |
| `version` | integer | yes | Policy schema version (currently 1) |
| `created_at` | string | yes | ISO 8601 creation timestamp |
| `rules` | array | no | Declarative rules. Evaluated in-process. |
| `executable` | string | no | Absolute path to a custom policy executable |
| `config` | object | no | Static configuration passed to the executable via `PolicyContext.policy_config` |
| `action` | string | yes | Currently `"deny"` only. Denied policies block the request. |

A policy MUST have at least one of `rules` or `executable`.

## PolicyContext

The JSON object available to policy evaluation:

```json
{
  "chain_id": "eip155:8453",
  "wallet_id": "3198bc9c-6672-5ab3-d995-4942343ae5b6",
  "api_key_id": "7a2f1b3c-4d5e-6f7a-8b9c-0d1e2f3a4b5c",
  "transaction": {
    "to": "0x742d35Cc6634C0532925a3b844Bc9e7595f2bD0C",
    "value": "100000000000000000",
    "raw_hex": "0x02f8...",
    "data": "0x"
  },
  "spending": {
    "daily_total": "50000000000000000",
    "date": "2026-03-22"
  },
  "timestamp": "2026-03-22T10:35:22Z"
}
```

## PolicyResult

```json
{ "allow": true }
```

```json
{ "allow": false, "reason": "Daily spending limit exceeded: 0.95 / 1.0 ETH" }
```

## Timeout and Failure Semantics

For custom executable policies only:

| Scenario | Result |
|----------|--------|
| Executable exits code 0, valid JSON on stdout | Use the PolicyResult as the verdict |
| Executable exits with non-zero code | **Deny** |
| Executable does not produce valid JSON on stdout | **Deny** |
| Executable does not exit within 5 seconds | **Deny**. Kill the process. |
| Executable not found or not executable | **Deny** |
| Unknown declarative rule type | **Deny**. Fail closed on unrecognized rules. |

The **default-deny** stance ensures that policy failures are never silently bypassed.

## Policy Attachment

Policies are attached to **API keys, not wallets**. An API key can have multiple policies attached. All must pass (AND semantics). Evaluation short-circuits on the first denial.

```bash
# Create a policy
ows policy create --file base-agent-limits.json

# Create an API key with wallet scope and policy attachment
ows key create --name "claude-agent" --wallet agent-treasury --policy base-agent-limits
# => ows_key_a1b2c3d4e5f6...  (shown once, store securely)
```

## Example: Custom Simulation Policy

A Python script that simulates an EVM transaction before allowing it:

```python
#!/usr/bin/env python3
"""Simulate transaction via eth_call before allowing."""
import json, sys, urllib.request

ctx = json.load(sys.stdin)
tx = ctx["transaction"]
rpc = {"eip155:8453": "https://mainnet.base.org"}.get(ctx["chain_id"])
if not rpc:
    json.dump({"allow": False, "reason": f"No RPC for {ctx['chain_id']}"}, sys.stdout)
    sys.exit(0)

payload = json.dumps({
    "jsonrpc": "2.0", "id": 1, "method": "eth_call",
    "params": [{"to": tx["to"], "value": hex(int(tx["value"])), "data": tx["data"]}, "latest"]
}).encode()
try:
    resp = json.load(urllib.request.urlopen(
        urllib.request.Request(rpc, data=payload, headers={"Content-Type": "application/json"}), timeout=4))
    if "error" in resp:
        json.dump({"allow": False, "reason": f"Reverted: {resp['error']['message']}"}, sys.stdout)
    else:
        json.dump({"allow": True}, sys.stdout)
except Exception as e:
    json.dump({"allow": False, "reason": str(e)}, sys.stdout)
```

## References

- [ERC-4337 Session Keys](https://eips.ethereum.org/EIPS/eip-4337)
