# OWS GitHub Repositories

Guide to the public repositories, their structure, and how they relate.

## Repository Map

```
open-wallet-standard/             ← GitHub organization
└── core                          ← Main repository (Rust + bindings + specs)
       168 ★  18 🍴  9 contributors  53 releases
       Rust 86.1% | TypeScript 10.2% | Python 2.5% | Shell 1.2%

moonpay/                          ← MoonPay GitHub organization
├── skills                        ← MoonPay Agents skills library
├── moonpay-sign                  ← Signing utilities
├── moonpay-demo-integrations     ← Integration examples
└── devops-challenge              ← Hiring challenge (unrelated)
```

## open-wallet-standard/core

**URL**: [github.com/open-wallet-standard/core](https://github.com/open-wallet-standard/core)

This is the canonical implementation and specification repository.

### Directory Structure

```
core/
├── ows/                    # Rust core library
│   ├── src/
│   │   ├── vault/          # Vault management (create, open, list, delete)
│   │   ├── wallet/         # Wallet operations (create, import, export)
│   │   ├── signer/         # Signing engine (sign, signAndSend, etc.)
│   │   ├── policy/         # Policy engine (declarative + custom)
│   │   ├── apikey/         # API key management (create, revoke, verify)
│   │   ├── crypto/         # Cryptographic primitives (AES, scrypt, HKDF)
│   │   ├── chain/          # Chain definitions and address derivation
│   │   └── lib.rs          # Public API surface
│   ├── Cargo.toml
│   └── tests/              # Integration tests
├── bindings/
│   ├── node/               # Node.js NAPI binding
│   │   ├── src/            # Rust → NAPI bridge code
│   │   ├── index.ts        # TypeScript API
│   │   ├── package.json    # @open-wallet-standard/core
│   │   └── tsconfig.json
│   └── python/             # Python CFFI binding
│       ├── src/            # Rust → CFFI bridge code
│       ├── open_wallet_standard/
│       │   └── __init__.py # Python API
│       └── pyproject.toml  # open-wallet-standard
├── docs/                   # Specification documents (Markdown)
│   ├── 01-storage-format.md
│   ├── 02-signing-interface.md
│   ├── 03-policy-engine.md
│   ├── 04-agent-access-layer.md
│   ├── 05-key-isolation.md
│   ├── 06-wallet-lifecycle.md
│   └── 07-supported-chains.md
├── website-docs/           # Source for docs.openwallet.sh
│   ├── quickstart.md
│   ├── sdk-cli.md
│   └── sdk-node.md
├── skills/
│   └── ows/                # OWS-native skills (MCP tool definitions)
│       ├── wallet_create.ts
│       ├── sign.ts
│       ├── sign_and_send.ts
│       └── index.ts
├── scripts/                # Build and release scripts
│   ├── build.sh
│   ├── release.sh
│   └── test.sh
├── .github/
│   └── workflows/          # CI/CD (build, test, publish)
├── LICENSE                 # MIT
├── README.md
└── Cargo.toml              # Workspace Cargo.toml
```

### Key Files

| File | Purpose |
|------|---------|
| `ows/src/lib.rs` | Public Rust API — the definitive interface |
| `bindings/node/index.ts` | Node.js SDK entry point |
| `bindings/python/open_wallet_standard/__init__.py` | Python SDK entry point |
| `docs/*.md` | The 7 specification documents |
| `skills/ows/index.ts` | OWS-native MCP skills |

### Build and Test

```bash
# Clone
git clone https://github.com/open-wallet-standard/core.git
cd core

# Build Rust core
cargo build --release

# Run tests
cargo test

# Build Node.js binding
cd bindings/node
npm install
npm run build

# Build Python binding
cd bindings/python
pip install -e .
```

### Release Process

Releases follow semantic versioning. Current version: **v1.0.0** (53 total releases including pre-releases).

Published artifacts:
- **npm**: `@open-wallet-standard/core` — includes prebuilt native binaries
- **PyPI**: `open-wallet-standard` — includes prebuilt wheels
- **crates.io**: `ows-cli` — Rust binary crate
- **GitHub Releases**: source archives + prebuilt binaries for macOS/Linux/Windows

## moonpay/skills

**URL**: [github.com/moonpay/skills](https://github.com/moonpay/skills)

The MoonPay Agents skills library. Contains all MCP-compatible skills.

### Structure

```
skills/
├── core/                   # Core wallet skills (delegates to OWS)
│   ├── wallet_create.ts
│   ├── wallet_import.ts
│   ├── sign.ts
│   └── ...
├── trading/                # Trading skills
│   ├── swap.ts
│   ├── bridge.ts
│   ├── quote.ts
│   └── ...
├── ramp/                   # On/off-ramp skills (MoonPay-specific)
│   ├── buy.ts
│   ├── sell.ts
│   └── ...
├── research/               # Research and analytics skills
│   ├── chain_info.ts
│   ├── tx_status.ts
│   ├── nft_list.ts
│   └── ...
├── x402/                   # x402 payment skills
│   ├── pay_request.ts
│   └── pay_discover.ts
├── shared/                 # Shared utilities, types, validation
├── mcp-server/             # MCP server implementation
│   ├── server.ts
│   ├── transport/
│   │   ├── stdio.ts
│   │   └── sse.ts
│   └── index.ts
└── package.json            # @moonpay/skills
```

### Relationship to OWS

The `core/` skills in moonpay/skills wrap the OWS SDK. They don't implement wallet logic themselves — they call `@open-wallet-standard/core` methods and expose them as MCP tools with proper input/output schemas.

## Other MoonPay Repositories

### moonpay/moonpay-sign

Signing utilities and helpers. Likely predates OWS and may contain legacy signing code.

### moonpay/moonpay-demo-integrations

Integration examples showing how to embed MoonPay services (widget, SDK) into web applications. Not directly related to OWS but provides context for MoonPay's broader platform.

## Contributing

The OWS repository accepts contributions under the MIT license. Key contribution areas as identified in the README:

- New chain support (add a chain family definition)
- SDK bindings for additional languages
- Policy engine plugins
- Documentation improvements
- Security audits and hardening

The primary maintainer is `@njdawn` (contributor to 53 releases).
