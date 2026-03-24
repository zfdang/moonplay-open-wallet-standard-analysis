# MoonPay Agents and Skills Ecosystem

How MoonPay implements the Open Wallet Standard to build an AI-native crypto agent platform.

## MoonPay Agents

MoonPay Agents is MoonPay's product line that implements OWS. Announced alongside the OWS launch on March 23, 2026, it provides:

1. **A non-custodial wallet** built on OWS — local key storage, AES-256-GCM encryption, policy-gated signing
2. **A skills framework** — modular, composable capabilities that agents can invoke
3. **MCP integration** — Model Context Protocol server that exposes skills to AI tools like Claude Desktop, Cursor, Windsurf, and others
4. **On/off-ramp access** — buy and sell crypto with fiat through MoonPay's existing infrastructure

The core thesis: AI agents need wallets, and those wallets should be open, local, and policy-constrained — not custodial black boxes.

## Architecture

```
┌─────────────────────────────────────────────┐
│            AI Agent (Claude, GPT, etc.)      │
└──────────────────┬──────────────────────────┘
                   │ MCP Protocol
                   ▼
┌─────────────────────────────────────────────┐
│            MoonPay MCP Server                │
│  ┌────────────────────────────────────────┐ │
│  │              Skills Layer               │ │
│  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐ │ │
│  │  │ Swap │ │ Send │ │Quote │ │ NFT  │ │ │
│  │  └──────┘ └──────┘ └──────┘ └──────┘ │ │
│  └────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────┐ │
│  │            OWS Runtime                  │ │
│  │  wallet · signer · policy · apikey     │ │
│  └────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
                   │
                   ▼
              ~/.ows/vault/
```

## Skills Library

Skills are the building blocks of MoonPay Agents. Each skill:
- Has a well-defined input/output schema
- Is independently versioned
- Can be composed with other skills
- Is exposed as an MCP tool

The skills are maintained in the public repository [github.com/moonpay/skills](https://github.com/moonpay/skills).

### Core Skills

| Skill | Description |
|-------|-------------|
| `wallet_create` | Create a new OWS wallet |
| `wallet_import` | Import an existing wallet |
| `wallet_export` | Export wallet material |
| `wallet_list` | List all wallets |
| `wallet_delete` | Delete a wallet |
| `address_derive` | Derive addresses for supported chains |
| `apikey_create` | Create an API key with policies |
| `apikey_revoke` | Revoke an API key |
| `sign` | Sign a transaction |
| `sign_and_send` | Sign and broadcast a transaction |
| `sign_message` | Sign an arbitrary message |
| `sign_typed_data` | Sign EIP-712 typed data |

### Trading & Markets

| Skill | Description |
|-------|-------------|
| `swap` | Swap one token for another (cross-chain supported) |
| `send` | Send tokens to an address |
| `bridge` | Bridge tokens between chains |
| `quote` | Get a price quote for a swap or bridge |
| `price` | Get token prices |
| `portfolio` | View portfolio balances across chains |
| `token_info` | Get detailed token information |
| `gas_estimate` | Estimate gas costs for a transaction |

### On/Off-Ramp (MoonPay-specific)

| Skill | Description |
|-------|-------------|
| `buy` | Buy crypto with fiat via MoonPay |
| `sell` | Sell crypto for fiat via MoonPay |
| `onramp_quote` | Get a buy quote |
| `offramp_quote` | Get a sell quote |
| `payment_methods` | List available payment methods |

### Research & Analytics

| Skill | Description |
|-------|-------------|
| `chain_info` | Get information about a blockchain |
| `tx_status` | Check transaction status |
| `tx_history` | View transaction history for an address |
| `contract_read` | Read from a smart contract |
| `ens_resolve` | Resolve ENS names to addresses |
| `nft_list` | List NFTs owned by an address |
| `nft_metadata` | Get NFT metadata |

### x402 Payments

| Skill | Description |
|-------|-------------|
| `pay_request` | Make a payment via the x402 protocol |
| `pay_discover` | Discover available merchants and endpoints via Bazaar directory |

## MCP Integration

MoonPay Agents uses the [Model Context Protocol](https://modelcontextprotocol.io/) to expose skills to AI agents.

### Starting the MCP Server

```bash
# Via MoonPay CLI
moonpay mcp start

# The server exposes all available skills as MCP tools
# Default: stdio transport (for Claude Desktop, Cursor)
# Optional: --transport sse --port 3001 (for HTTP-based clients)
```

### Claude Desktop Configuration

```json
{
  "mcpServers": {
    "moonpay": {
      "command": "moonpay",
      "args": ["mcp", "start"]
    }
  }
}
```

### Cursor Configuration

```json
{
  "mcpServers": {
    "moonpay": {
      "command": "moonpay",
      "args": ["mcp", "start"]
    }
  }
}
```

### Example Interaction

User says to Claude: "Swap 0.1 ETH for USDC on Base"

Claude invokes the `swap` skill:

```json
{
  "tool": "swap",
  "arguments": {
    "fromToken": "eip155:8453:native",
    "toToken": "eip155:8453:0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
    "amount": "0.1",
    "walletId": "default"
  }
}
```

The skill:
1. Gets a quote from a DEX aggregator
2. Constructs the swap transaction
3. Calls OWS `signAndSend` with the agent's API key
4. Policy engine evaluates the request (is `eip155:8453` allowed? Is the amount within limits?)
5. If approved, signs and broadcasts the transaction
6. Returns the transaction hash to Claude

## Agent Security Model

MoonPay Agents inherits OWS security properties:

| Concern | Mitigation |
|---------|-----------|
| Agent spends too much | Attach spending-limit policies to the API key |
| Agent uses wrong chain | Attach `allowed_chains` policy to restrict to specific networks |
| API key stolen | Revoke via `ows apikey revoke`; delete the API key file |
| Agent sees private keys | Never — keys are derived inside the signer and zeroized after use |
| Malicious skill | Skills don't have direct key access; all signing goes through the OWS policy engine |
| Session expiry | Set `expires_at` on the API key |

## OWS Skills Directory

The `skills/ows/` directory inside the `open-wallet-standard/core` repo contains OWS-native skills that any implementation can use. These are distinct from MoonPay-specific skills:

```
skills/ows/
├── wallet_create.ts
├── wallet_import.ts
├── wallet_list.ts
├── address_derive.ts
├── sign.ts
├── sign_and_send.ts
├── sign_message.ts
├── sign_typed_data.ts
├── apikey_create.ts
├── apikey_revoke.ts
└── index.ts
```

These skills follow a standard interface:
```typescript
interface Skill {
  name: string;
  description: string;
  inputSchema: JSONSchema;
  outputSchema: JSONSchema;
  execute(input: unknown): Promise<unknown>;
}
```

## Relationship Between OWS and MoonPay

| Aspect | OWS | MoonPay Agents |
|--------|-----|----------------|
| Governance | Open standard, MIT license | MoonPay product |
| Scope | Wallet spec + signing + policies | Full agent platform |
| Skills | Core wallet skills only | Core + trading + on/off-ramp + research |
| On/Off-ramp | Not included | MoonPay fiat <→ crypto |
| MCP server | Reference implementation | Production implementation |
| Commercial | Free, open source | Freemium (MoonPay API keys for ramp services) |

OWS is the open standard; MoonPay Agents is MoonPay's flagship implementation that adds commercial services on top.
