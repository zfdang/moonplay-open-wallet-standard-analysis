# MoonPay / OWS Risk Analysis

## Scope and Evidence Boundaries

This document synthesizes the public OWS materials reviewed in this repository and reorganizes them into a security-boundary and risk-focused analysis.

One boundary matters throughout:

- **Public OWS specification / reference implementation materials** explicitly describe the local `~/.ows/` vault, `scrypt`, `HKDF-SHA256`, `AES-256-GCM`, policy behavior, and key-isolation posture
- **Public MoonPay CLI / MoonPay Agents materials** explicitly describe a non-custodial local product using **OS keychain encryption**
- therefore, when this document discusses exact file layouts, KDFs, ciphers, token wrapping, or decryption order, it is referring by default to the **public OWS reference-implementation model**
- those details should **not** be treated as proof that the MoonPay product uses the exact same internal implementation

That source boundary is also documented in [docs/ows-public-source-review.md](docs/ows-public-source-review.md) and [docs/ows-moonpay-agents-and-skills.md](docs/ows-moonpay-agents-and-skills.md).

## Executive Summary

The public OWS model has several real strengths:

- local encrypted-at-rest wallet storage
- scoped agent access through API tokens
- policy evaluation before decryption on the intended OWS code path
- short-lived decryption and explicit zeroization

But its most important security boundaries are also its most important limits:

- the current public implementation posture is **in-process**, not enclave-isolated
- agent authorization is primarily **bearer-token capability**, not strong agent identity
- policy is mainly an **OWS code-path boundary**, not a ciphertext-level cryptographic boundary
- the system assumes local OS and filesystem trust much more than remote, tamper-resistant infrastructure

So the main real-world risk is not "cryptographically cracking the disk ciphertext." It is:

- token leakage
- vault-read exposure
- owner-credential misuse
- in-process memory compromise
- narrow policy expressiveness
- local-only operational trust assumptions

## OWS Security Boundaries

### 1. Encrypted-at-Rest Boundary

Under the public OWS reference model:

- owner wallet secrets are stored in `~/.ows/wallets/<wallet-id>.json`
- the encrypted payload is either mnemonic entropy or a raw private key
- wallet files are encrypted with a key derived from `scrypt(passphrase)` and wrapped with `AES-256-GCM`
- agent API key files are stored in `~/.ows/keys/<key-id>.json`
- those key files contain a second encrypted copy of the wallet secret, wrapped under a key derived from `HKDF-SHA256(token)`

What this boundary protects:

- a disk-only attacker without the owner passphrase or raw API token

What it does not fully protect:

- weak passphrases
- token leakage
- same-user local process compromise
- host-process compromise after decryption

### 2. Credential Boundary: Owner vs Agent

OWS draws a sharp line between:

- **owner credential**: full wallet access, no policy checks
- **agent credential (`ows_key_...`)**: wallet-scoped, policy-constrained access

This is a meaningful security boundary, but it also means:

- owner credentials are effectively "sudo"
- agent safety depends on never letting untrusted automation obtain the owner passphrase

### 3. Policy Boundary

On the normal OWS path, the agent flow is:

1. resolve API key by token
2. check scope and expiry
3. evaluate attached policies
4. only then decrypt the wallet secret
5. sign and wipe

That is a good design direction, but it is important to describe the boundary correctly:

- policy is enforced **by the OWS implementation path**
- policy is **not** cryptographically embedded into the `wallet_secrets` ciphertext

So if an attacker has all of the following:

- the raw `ows_key_...` token
- read access to the relevant `~/.ows/keys/*.json`
- the ability to run custom code

then they can theoretically implement `HKDF-SHA256 + AES-256-GCM` themselves and decrypt `wallet_secrets[wallet_id]` directly, bypassing OWS policy evaluation.

This is why policy should be treated as a **logical security boundary**, not a hard cryptographic one.

### 4. Process Boundary

The current public implementation posture is **in-process signing**:

- the OWS core is linked into the calling process
- the signer and agent/app share an address space
- hardening includes `mlock()`, zeroization, anti-ptrace, and anti-coredump

This reduces exposure, but it is not the same as a subprocess enclave or TEE boundary.

So the current process boundary is:

- better than leaving plaintext keys on disk
- weaker than isolating signing into a dedicated trust boundary

### 5. Local OS / Filesystem Boundary

OWS assumes a local-machine trust model:

- the vault is local
- file permissions should be strict (`700` / `600`)
- access layers are local by design
- local-service mode must authenticate callers or OS principals

This means OWS relies heavily on:

- OS user isolation
- filesystem permissions
- local process trust
- correct secret-delivery hygiene

It is not designed as a remote multi-tenant custody system.

### 6. Product Boundary: OWS vs MoonPay

OWS and MoonPay should not be collapsed into a single security claim.

What the public sources support:

- OWS publicly defines storage, signing, policy, lifecycle, and conformance behavior
- MoonPay publicly says MoonPay Agents implements OWS
- MoonPay publicly says MoonPay CLI uses local wallets with OS keychain encryption

What the public sources do **not** prove:

- that MoonPay uses the exact same local file layout
- that MoonPay uses the exact same KDF/cipher combination
- that MoonPay exposes the exact same runtime trust boundaries as the public OWS reference implementation

That leaves a real assurance gap for product-level security review.

## Major Security Risks

### High Risk 1: Token + vault access can bypass the policy path

Impact:

- an attacker who obtains the raw token and the relevant key file can recover the wrapped wallet secret without going through OWS policy evaluation
- in that scenario, policy is only an enforced check in the intended OWS path, not a hard boundary at the ciphertext layer

Why:

- the token carries both authentication and decryption power
- `key.json` already contains the encrypted wallet-secret copy associated with that token

### High Risk 2: The current public implementation is in-process

Impact:

- malicious plugins, dependency injection, RCE, or in-process memory scraping may capture secrets during the short window after decryption and before zeroization
- static-at-rest encryption does not solve this

Why:

- the current public posture is not a TEE / enclave
- the signer and caller share an address space

### High Risk 3: Owner credentials bypass all policies

Impact:

- if an agent, script, MCP server, or developer tool obtains the owner passphrase, it gains effectively unrestricted wallet control
- this completely bypasses agent-mode scope and policy controls

Why:

- the public OWS model explicitly says owner mode does not pass through policy

### High Risk 4: Secret-delivery mechanisms can leak credentials

Impact:

- `OWS_PASSPHRASE`, `OWS_PRIVATE_KEY`, `OWS_MNEMONIC`, and similar inputs may leak through `/proc`, crash dumps, child-process inheritance, shell history, CI logs, or observability tooling
- `--show-mnemonic`, export flows, and stdin-based import/export paths can also leak operationally

Why:

- the weakest part of many local wallet systems is not the storage cipher but how secrets are supplied to the process

### Medium Risk 5: Weak passphrases still create offline guessing exposure

Impact:

- if an attacker gets a wallet file and the passphrase is weak, offline guessing becomes realistic enough to matter

Why:

- `scrypt` raises guessing cost but does not eliminate brute-force risk
- wallet-file strength ultimately depends in part on passphrase quality

### Medium Risk 6: Built-in policy capabilities are narrow

Impact:

- teams relying only on built-in declarative rules usually get little more than chain restriction and expiry
- high-value controls such as spend caps, destination allowlists, rate limits, and simulation checks are not built-in standard rules

Why:

- the public standard stabilizes only `allowed_chains` and `expires_at` as built-in declarative rule types

### Medium Risk 7: Executable policy increases flexibility, but also the attack surface

Impact:

- executable policies can introduce supply-chain risk, path hijacking, logic errors, poisoned dependencies, or incorrect allow decisions

Why:

- executable policy is an external program, not a tightly constrained policy DSL

### Medium Risk 8: API-key sprawl increases the number of encrypted secret copies

Impact:

- each API key stores its own re-encrypted wallet-secret copy
- the more agents there are, the more secret wrappers exist
- stale tokens, orphaned key files, and incomplete cleanup expand the attack surface

Why:

- the agent model creates per-token wrapped copies rather than using the owner wallet secret directly for every request

### Medium Risk 9: No standardized nonce coordination for shared wallets

Impact:

- multiple agents using the same wallet concurrently may trigger nonce collisions, transaction replacement, duplicate signing, or business-level idempotency failures

Why:

- the public signing interface explicitly leaves nonce coordination to higher layers

### Medium Risk 10: Local audit logs are operationally useful but weak as standalone forensic controls

Impact:

- append-only is true from the OWS application's point of view
- an attacker with higher OS privileges may still tamper with, delete, or suppress local logs

Why:

- remote tamper-resistant audit is not part of the core public OWS standard

### Medium Risk 11: There is a transparency gap between MoonPay product claims and OWS reference detail

Impact:

- public MoonPay claims are not enough to verify that MoonPay has the same storage, key-handling, and runtime boundaries as the public OWS reference implementation

Why:

- MoonPay public materials do not expose the same level of implementation detail as the OWS public reference docs

## Recommended Minimum Security Baseline

If OWS / MoonPay Agents is used for real-value environments, the minimum sensible baseline should include:

1. **Agents get API tokens, never the owner passphrase**
2. **Untrusted agents must not hold both the raw token and read access to `~/.ows/`**
3. **Use one agent / one environment / one wallet / one token wherever possible**
4. **Set short token expiries and support fast revocation**
5. **Use strong passphrases and avoid long-lived secret delivery through environment variables**
6. **Use executable policies for production agents** to enforce amount, destination, method, chain, time window, and simulation requirements
7. **For high-value wallets, do not expose in-process OWS directly to untrusted agents**; prefer subprocess or local-service isolation
8. **Add external nonce coordination and audit-log forwarding**
9. **Treat the MoonPay product as a separate audit target** rather than relying on the OWS reference model alone

## Final Judgment

If we only use the public OWS materials, the balanced conclusion is:

- the encrypted-at-rest design is not obviously weak
- the policy-before-decrypt flow is the right intended control path
- the most important security issues are not classical cryptographic breaks
- the most important security issues are token leakage, vault-read exposure, owner-credential misuse, in-process execution, narrow built-in policy power, and local trust-boundary assumptions

So the best compact description of OWS is:

- **OWS is strongest as a local encrypted wallet and signing standard with scoped agent access on the intended code path**
- **OWS is weakest when untrusted agent code can combine raw tokens, vault access, and arbitrary local execution**

## References

- [docs/ows-storage-format.md](docs/ows-storage-format.md)
- [docs/ows-wallet-lifecycle.md](docs/ows-wallet-lifecycle.md)
- [docs/ows-policy-engine.md](docs/ows-policy-engine.md)
- [docs/ows-signing-interface.md](docs/ows-signing-interface.md)
- [docs/ows-agent-access-layer.md](docs/ows-agent-access-layer.md)
- [docs/ows-key-isolation-and-security.md](docs/ows-key-isolation-and-security.md)
- [docs/ows-specification-conformance-and-security.md](docs/ows-specification-conformance-and-security.md)
- [docs/ows-moonpay-agents-and-skills.md](docs/ows-moonpay-agents-and-skills.md)
- [docs/ows-public-source-review.md](docs/ows-public-source-review.md)
