# MoonPay Open Wallet Standard Analysis

This repository is a source-checked analysis of the public Open Wallet Standard (OWS) materials and the public MoonPay Agents / MoonPay CLI materials around them.

## Review Status

This repository was reread and revised against public sources on **2026-03-24**.

Authority order used in this review:

1. the numbered OWS docs in `open-wallet-standard/core/docs/`
2. `00-specification.md` and `08-conformance-and-security.md`
3. the public `open-wallet-standard/core` README and `openwallet.sh`
4. MoonPay public product pages, Help Center articles, newsroom posts, and GitHub org pages for MoonPay-specific claims

The detailed review notes live in [ows-public-source-review.md](docs/ows-public-source-review.md).

## Important Public-Source Notes

- The public OWS sources are useful but not perfectly aligned. Where the overview pages, quickstart, and numbered docs conflict, this repository now follows the numbered docs and the conformance/security docs.
- `docs.openwallet.sh` currently exposes the numbered docs through `07-supported-chains.md` in the top navigation, while the public GitHub repo and README also include `08-conformance-and-security.md`.
- The public quickstart shows a `Signing Enclave (isolated proc)` diagram, but `05-key-isolation.md` says current implementations are **in-process** and presents subprocess isolation as a future profile.
- The public OWS README advertises a `Policy Engine Implementation Guide`, but that document was not directly accessible during this review. This repository now treats it as an advertised reference doc, not as a stable public source for module-level claims.
- `07-supported-chains.md` defines 9 chain families including Spark, while the current CLI / SDK examples show 8 auto-derived chain accounts. This repository treats the numbered chain doc as normative and the SDK docs as current implementation examples.
- MoonPay Agents publicly says it implements OWS, but MoonPay Help Center docs describe MoonPay CLI wallets as local HD wallets with **OS keychain encryption**, while the public OWS reference docs describe a `~/.ows/` AES-256-GCM / scrypt vault. This repository therefore keeps a strict boundary between **the OWS reference implementation** and **MoonPay's product implementation**.

## Recommended Reading Order

1. [ows-public-source-review.md](docs/ows-public-source-review.md)
   Review findings, source-precedence rules, public-source inconsistencies, and the coverage matrix for this repository.

2. [ows-overview.md](docs/ows-overview.md)
   What OWS is, the six design principles on `openwallet.sh`, what can be stated safely about its relationship to MoonPay, and which homepage claims are marketing-level versus spec-level.

3. [ows-architecture.md](docs/ows-architecture.md)
   A synthesis of the public architecture story across the homepage, README, policy docs, access-layer docs, and key-isolation docs.

4. [ows-storage-format.md](docs/ows-storage-format.md)
   The normative vault layout, wallet and API-key file structures, filesystem permissions, crypto envelope details, and backward compatibility rules.

5. [ows-key-isolation-and-security.md](docs/ows-key-isolation-and-security.md)
   Current in-process hardening, threat model, key lifecycle, short-lived key caching, and the future subprocess-enclave profile.

6. [ows-signing-interface.md](docs/ows-signing-interface.md)
   The normative signing operations, error model, and where the public overview page diverges from `02-signing-interface.md`.

7. [ows-policy-engine.md](docs/ows-policy-engine.md)
   Owner versus agent mode, API-key cryptography, declarative rules, executable policies, default-deny behavior, and current file-format limits.

8. [ows-agent-access-layer.md](docs/ows-agent-access-layer.md)
   The optional local access profiles and the interoperability constraints they must preserve.

9. [ows-wallet-lifecycle.md](docs/ows-wallet-lifecycle.md)
   Creation, import, export, backup, recovery, deletion, rotation, and discovery.

10. [ows-supported-chains.md](docs/ows-supported-chains.md)
    Chain families, identifiers, derivation paths, shorthand aliases, and the out-of-scope boundary around endpoint selection.

11. [ows-specification-conformance-and-security.md](docs/ows-specification-conformance-and-security.md)
    Document classes, conformance targets, optional features, extension rules, and the concrete security requirements from the public conformance doc.

12. [ows-sdk-cli-reference.md](docs/ows-sdk-cli-reference.md)
    The public reference-implementation surfaces: `ows`, `@open-wallet-standard/core`, and `open-wallet-standard`, including the current CLI and SDK drift points.

13. [ows-reference-implementation-guide.md](docs/ows-reference-implementation-guide.md)
    How to read the quickstart and SDK docs correctly, and why they should not be treated as normative spec text.

14. [ows-github-repos.md](docs/ows-github-repos.md)
    What is verifiably public in `open-wallet-standard/core` and MoonPay's GitHub organization, and which path-level claims were removed because they were not directly verifiable.

15. [ows-moonpay-agents-and-skills.md](docs/ows-moonpay-agents-and-skills.md)
    MoonPay Agents, MoonPay CLI, MCP setup, published skill names, and the product-versus-standard boundary.

16. [ows-x402-payments-integration.md](docs/ows-x402-payments-integration.md)
    What the public OWS and MoonPay docs actually establish about x402, paid requests, Bazaar discovery, and why payments remain outside core-spec conformance.

## Coverage Map

Reading the repository in the order above covers:

- the OWS homepage narrative and the numbered spec documents
- storage, signing, policy, lifecycle, chain, and conformance requirements
- access-layer and key-isolation guidance
- the public CLI, Node, and Python reference surfaces
- the public GitHub repository surface for OWS and MoonPay
- MoonPay Agents, MoonPay CLI, MCP integration, and MoonPay's published skill surface
- x402 and payment/discovery features as implementation-layer capabilities rather than core-spec requirements
- the current set of publicly visible documentation conflicts and how this repository resolves them

## Primary Public Sources

- `https://openwallet.sh/`
- `https://docs.openwallet.sh/`
- `https://github.com/open-wallet-standard/core`
- `https://www.moonpay.com/agents`
- `https://support.moonpay.com/en/collections/1373008-ai-agents-and-cli-tools`
- `https://github.com/moonpay`
