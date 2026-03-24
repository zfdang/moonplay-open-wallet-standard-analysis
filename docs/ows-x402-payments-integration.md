# OWS x402 Payments Integration

How OWS integrates the x402 payment protocol for agent-to-agent and agent-to-service payments.

## What is x402?

x402 is a payment protocol inspired by the HTTP 402 "Payment Required" status code. It defines a standard way for:
- A service to request payment before fulfilling a request
- An agent (or user) to make that payment on-chain
- The service to verify payment and complete the request

OWS implements x402 as a first-class payment flow, allowing AI agents to pay for services autonomously, subject to the same policy constraints as any other signing operation.

## Payment Flow

```
┌─────────┐                   ┌──────────┐                ┌──────────────┐
│  Agent   │                   │  Service  │                │  Blockchain  │
└────┬─────┘                   └─────┬────┘                └──────┬───────┘
     │                               │                            │
     │  1. Request resource          │                            │
     │──────────────────────────────▶│                            │
     │                               │                            │
     │  2. HTTP 402 + payment terms  │                            │
     │◀──────────────────────────────│                            │
     │                               │                            │
     │  3. Construct payment tx      │                            │
     │  4. OWS sign (policy check)   │                            │
     │  5. Broadcast tx              │                            │
     │───────────────────────────────────────────────────────────▶│
     │                               │                            │
     │  6. tx hash                   │                            │
     │◀──────────────────────────────────────────────────────────│
     │                               │                            │
     │  7. Retry request + tx proof  │                            │
     │──────────────────────────────▶│                            │
     │                               │  8. Verify payment on-chain│
     │                               │───────────────────────────▶│
     │                               │                            │
     │                               │  9. Confirmed              │
     │                               │◀───────────────────────────│
     │                               │                            │
     │  10. Resource delivered       │                            │
     │◀──────────────────────────────│                            │
     │                               │                            │
```

## OWS Skills

### `ows pay request`

Makes a payment to a service endpoint.

```bash
ows pay request \
  --to <recipient-address> \
  --amount <amount> \
  --asset <asset-id> \
  --chain <chain-id> \
  --wallet <wallet-id> \
  --api-key <api-key-token>
```

Parameters:
- `--to`: Recipient address (CAIP-10 account or raw address)
- `--amount`: Payment amount in token units (e.g., "0.01" for 0.01 USDC)
- `--asset`: Asset identifier (e.g., `eip155:8453:0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` for USDC on Base)
- `--chain`: Chain to transact on
- `--wallet`: Wallet ID to pay from
- `--api-key`: API key for agent-mode signing

The payment goes through the standard OWS signing pipeline:
1. Construct a transfer transaction
2. Evaluate policies attached to the API key (allowed chains? spending limits? custom policy?)
3. If approved, sign and broadcast
4. Return transaction hash

### `ows pay discover`

Discovers available services and their payment terms via the Bazaar directory.

```bash
ows pay discover [--category <category>] [--chain <chain-id>]
```

Returns a list of registered services with:
- Service name and description
- Accepted chains and tokens
- Price per request or subscription terms
- Endpoint URL

## Bazaar Directory

Bazaar is a decentralized directory service where:
- **Service providers** register their endpoints and payment terms
- **Agents** discover services they can pay for

This enables an agent economy where AI agents can autonomously find and pay for services — data APIs, compute resources, other agents' capabilities — without human intervention.

## Policy Integration

x402 payments are subject to the same OWS policy engine as any other signing operation. This is critical for agent safety:

```json
{
  "version": "1.0.0",
  "rules": {
    "allowed_chains": ["eip155:8453"],
    "max_transaction_value": {
      "asset": "eip155:8453:0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
      "amount": "10.00"
    },
    "daily_spending_limit": {
      "asset": "eip155:8453:0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
      "amount": "100.00"
    }
  }
}
```

With this policy, an agent can make x402 payments of up to $10 per transaction and $100 per day in USDC on Base — but nothing else. The owner retains full control over the agent's spending behavior.

## Security Considerations

| Concern | Mitigation |
|---------|-----------|
| Agent overspends | Spending limit policies (per-tx and daily) |
| Payment to wrong recipient | Custom policy can whitelist recipient addresses |
| Replay attacks | Each transaction has a unique nonce; blockchain prevents replay |
| Service doesn't deliver | Out of scope for OWS; resolved at the application layer |
| Man-in-the-middle | HTTPS for service communication; on-chain verification for payment |
