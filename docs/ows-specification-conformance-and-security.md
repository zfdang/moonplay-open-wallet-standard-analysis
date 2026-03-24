# OWS Specification, Conformance, and Security

This document covers the public material that was missing from the repository's first pass: the scope-setting specification document and the conformance/security document.

It also resolves one public-source mismatch: the docs site navigation currently omits `08-conformance-and-security.md`, but the GitHub repo and public README include it. This file treats `08-conformance-and-security.md` as part of the public normative surface.

## Document Classes

The public `00-specification.md` defines how to read the rest of OWS.

### Normative core

The following are defined as the **normative core**:

- `01-storage-format.md`
- `02-signing-interface.md`
- `03-policy-engine.md`
- `06-wallet-lifecycle.md`
- `07-supported-chains.md`
- `08-conformance-and-security.md`

These are the documents that define interoperable behavior.

### Optional profiles

The public spec says:

- `04-agent-access-layer.md`
- `05-key-isolation.md`

describe optional access and deployment profiles. They are part of the architecture, but they do not require a specific package manager, programming language, or transport.

### Non-normative reference implementation docs

The public spec explicitly classifies these as non-normative:

- `quickstart.md`
- `sdk-cli.md`
- `sdk-node.md`
- `sdk-python.md`
- `policy-engine-implementation.md`

If they conflict with a normative core doc, the normative core doc wins.

For this repository's review process, one extra caveat applies: `policy-engine-implementation.md` is publicly advertised but was not directly accessible during this review, so this repository does not use it as the basis for module-level implementation claims.

## Conformance Targets

The public specification defines these conformance targets:

1. Storage conformance
2. Signing conformance
3. Policy conformance
4. Lifecycle conformance
5. Chain conformance

It also says an implementation is conforming only for the parts it fully implements, and must not claim blanket OWS compliance if it only implements a subset.

The public example format is:

```text
OWS Storage + Signing + EVM Chain Profile
```

That is stricter and more precise than a vague "OWS compatible" claim.

## Optional Features

`00-specification.md` marks the following as optional unless a calling profile requires them:

- `signAndSend`
- `signTypedData`
- executable policies
- subprocess or enclave-style isolation
- shorthand aliases in interactive CLI contexts

If omitted, an implementation must fail clearly and must not silently degrade to weaker behavior.

## Extension Rules

The public spec permits extension in controlled places:

- new chain families may be added if they define a stable canonical identifier, deterministic derivation path, and address encoding rule
- policy engines may add implementation-specific declarative rule types, but they must namespace them and reject unknown unnamespaced rule types
- wallet, API key, and policy files may include additional metadata fields, and implementations must preserve unknown fields on non-destructive updates

Extensions must not redefine existing required fields.

## Out of Scope

The public `00-specification.md` explicitly marks these topics out of scope for the core standard:

- package names and distribution channels
- public RPC endpoint selection
- hosted wallet services and remote custody
- on-chain service payment flows
- UI, CLI ergonomics, and installer behavior

This single section is critical for reviewing the rest of the repository, because it tells us which claims belong in OWS analysis and which belong only in reference implementation notes.

## Conformance Claims

The public `08-conformance-and-security.md` requires that an implementation claiming OWS conformance declare the profiles it supports.

Minimum format:

```text
OWS <supported profiles>
```

The public examples include:

- `OWS Storage + Signing + Policy + EVM Chain Profile`
- `OWS Storage + Signing + Lifecycle + Solana Chain Profile`

An implementation must not claim complete OWS conformance if it omits required behavior from a profile it advertises.

## Interoperability Artifacts

The conformance doc says conforming implementations should ship or consume machine-readable test vectors for:

- wallet file decryption and encryption
- API key file resolution and token verification
- policy rule evaluation
- chain-specific address derivation
- transaction and message signing

It also says that interoperable implementations must preserve:

- consistent wallet-file parsing and validation
- consistent API-key resolution by token hash
- consistent policy allow/deny outcomes for the same `PolicyContext`
- canonical chain and account identifiers without lossy conversion

## Minimum Test Vector Set

The public minimum test set includes at least:

1. one wallet file using scrypt + AES-256-GCM
2. one API key file using HKDF-SHA256 + AES-256-GCM
3. one declarative policy that allows a supported chain
4. one declarative policy that denies by expiration
5. one signing vector per supported chain family
6. one negative vector for each documented error code

If executable policies are supported, the public doc also recommends fixtures for:

- successful executable evaluation
- denial on executable failure
- denial on malformed output

## Error Consistency

The conformance doc says implementations must preserve the meanings defined by the signing-interface document.

They may add metadata, but they must not:

- turn policy denial into generic auth failure
- collapse unsupported-chain errors into malformed-input errors
- treat expired API keys as missing keys

## Security Requirements

The public conformance/security doc organizes security into four concrete areas.

### Secret Material

Implementations must:

- decrypt wallet or API-key-backed secrets only for the duration of the operation
- zeroize decrypted mnemonic, private key, derived key, and KDF output buffers after use
- avoid writing decrypted secret material to logs, telemetry, or audit records

They should use hardened memory and process protections when available.

### Credential Handling

Implementations must:

- treat owner credentials and API tokens as secrets
- avoid echoing credentials in logs or human-readable errors
- verify API-token scope and policy attachments before any token-backed secret is decrypted

They should avoid environment variables for long-lived secrets unless the environment is trusted and documented.

### Policy Enforcement

Implementations must:

- evaluate built-in rules deterministically
- short-circuit on denial when the model requires it
- deny if an executable policy exits unsuccessfully or returns malformed output

They must not provide a fallback path that bypasses token-attached policy evaluation.

### Audit Logging

The public doc says audit logs must be append-only from the point of view of the OWS implementation.

Audit records should include:

- operation type
- wallet identifier
- chain identifier
- API key identifier when applicable
- allow or deny outcome
- timestamp

Audit records must not contain raw passphrases, API tokens, mnemonics, or private keys.

## Reference Guidance

The public conformance doc makes one more important distinction: `05-key-isolation.md` provides optional implementation guidance for hardening and subprocess isolation, but interoperability depends on the externally visible behavior of the core spec.

That is the right way to separate conformance requirements from implementation hardening advice.

## Primary Sources

- `https://github.com/open-wallet-standard/core/blob/main/docs/00-specification.md`
- `https://github.com/open-wallet-standard/core/blob/main/docs/08-conformance-and-security.md`
- `https://docs.openwallet.sh/`
