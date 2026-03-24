# OWS Agent Access Layer

How OWS functionality may be exposed to applications, agents, CLIs, and local services without changing the core semantics defined by the numbered specs.

This document follows `04-agent-access-layer.md` for the normative profile shape. The homepage and marketing materials sometimes name concrete surfaces like MCP or REST, but the numbered access-layer doc keeps the contract intentionally abstract.

## Purpose

The public agent-access document is explicit about scope:

- the core spec defines stored artifacts, signing semantics, policy evaluation, wallet lifecycle behavior, and chain identifiers
- implementations may expose those capabilities through different **local access layers**
- those surfaces may differ, but they **must preserve the core OWS semantics**

This makes `04-agent-access-layer.md` an architectural profile document, not a package-level API reference.

## Required Access Capabilities

The spec says a conforming access layer must preserve these capabilities:

| Capability | Requirement |
|---|---|
| Wallet selection | MUST identify the target wallet unambiguously by ID or implementation-defined stable alias |
| Chain selection | MUST resolve the request to a canonical chain identifier before signing |
| Credential handling | MUST distinguish owner credentials from API tokens without ambiguity |
| Policy enforcement | MUST evaluate applicable policies before any token-backed secret is decrypted |
| Error propagation | MUST surface core errors without rewriting a denial into a success or silent fallback |
| Secret handling | MUST NOT expose decrypted mnemonic or private key material to the caller unless an explicit export operation is invoked |

These are the real interoperability requirements of the access layer.

## Abstract Operations

The public spec gives an abstract operation set. It is intentionally generic and does not require any specific package names or CLI verbs.

```text
createWallet(request) -> WalletDescriptor
importWallet(request) -> WalletDescriptor
listWallets(request?) -> WalletDescriptor[]
getWallet(request) -> WalletDescriptor
deleteWallet(request) -> DeleteResult
sign(request) -> SignResult
signAndSend(request) -> SignAndSendResult
signMessage(request) -> SignMessageResult
signTypedData(request) -> SignMessageResult
createPolicy(request) -> PolicyDescriptor
listPolicies(request?) -> PolicyDescriptor[]
getPolicy(request) -> PolicyDescriptor
deletePolicy(request) -> DeleteResult
createApiKey(request) -> ApiKeyCreationResult
listApiKeys(request?) -> ApiKeyDescriptor[]
revokeApiKey(request) -> DeleteResult
```

That list is narrower and more precise than inventing extra contract-level operations such as backup, restore, inspect, chain listing, or address derivation.

## Credential Semantics

The access-layer spec restates the policy-engine distinction:

- **owner credential**: unlocks the wallet directly and bypasses policy checks
- **API token**: resolves to an API key, enforces wallet scope, and evaluates attached policies before decryption

If a surface accepts a single generic credential field, it must use deterministic credential-type detection and must not guess in a way that could weaken enforcement.

## Access Profiles

The public spec describes three access profiles.

### Profile A: In-Process Binding

The caller links directly against an OWS implementation in the same process.

Requirements called out by the spec:

- MUST preserve all core signing and policy semantics
- MUST zeroize decrypted secret material as soon as the operation completes
- SHOULD document that the application and signer share an address space

### Profile B: Local Subprocess

The caller spawns an OWS child process per operation or per session.

Requirements called out by the spec:

- MUST provide authenticated request input to the child process
- MUST ensure policy evaluation happens before token-backed secrets are decrypted
- SHOULD use structured request and response payloads

### Profile C: Local Service

The caller communicates with a loopback-only daemon or local RPC endpoint.

Requirements called out by the spec:

- MUST bind only to local interfaces unless a stronger trust boundary is explicitly documented
- MUST authenticate callers or operating-system principals before performing owner or token-backed actions
- MUST map remote method names back to the core OWS operations without changing their semantics

## Cross-Layer Consistency

If an implementation offers multiple access layers, all of them must agree on:

- wallet and API key lookup behavior
- policy evaluation order
- canonical error codes
- chain identifier normalization
- audit-log side effects

The public spec also says an implementation must not make one surface stricter or weaker than another for the same operation unless the difference is explicitly documented as surface-specific validation.

## What the Access Layer Does Not Standardize

The public document explicitly excludes these details from the standard:

- package names such as npm or PyPI artifacts
- shell installer commands
- generated client stubs
- framework-specific wrappers

Those belong in the reference implementation docs, not in the normative access-layer contract.

## Relationship to the Rest of OWS

The practical reading is:

- `02-signing-interface.md` defines the signing behavior
- `03-policy-engine.md` defines policy evaluation and API-token semantics
- `04-agent-access-layer.md` defines how those behaviors may be exposed safely through local surfaces

That is why the access-layer document is important, but also why it should not be turned into an invented SDK or CLI API.

## Primary Sources

- `https://github.com/open-wallet-standard/core/blob/main/docs/04-agent-access-layer.md`
- `https://openwallet.sh/`
