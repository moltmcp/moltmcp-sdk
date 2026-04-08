# @moltmcp/sdk

> TypeScript SDK for integrating MoltMCP autonomous payments into AI agents and applications.

[![npm version](https://img.shields.io/npm/v/@moltmcp/sdk)](https://www.npmjs.com/package/@moltmcp/sdk)
[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Base](https://img.shields.io/badge/chain-Base-0052FF?logo=coinbase)](https://base.org)

---

## Installation

```bash
npm install @moltmcp/sdk
# or
yarn add @moltmcp/sdk
```

---

## Quick Start

```ts
import { MoltMCPClient } from '@moltmcp/sdk'

const client = new MoltMCPClient({
  baseRpc: 'https://mainnet.base.org',
  wallet: await loadAgentWallet(),
  policies: {
    maxPerTransaction: 100_000, // 100 USDC (in micro-units)
    dailyLimit: 1_000_000       // 1000 USDC
  }
})

// Agent encounters HTTP 402 — pay automatically
const result = await client.payForResource({
  url: 'https://api.weather.com/premium/forecast',
  parameters: { location: 'SF', days: 7 }
})

console.log(`Payment finalized: ${result.hash}`)
```

---

## Features

- **Intent-based payment API** — describe what you need, SDK handles the rest
- **Automatic MPP negotiation** — parses HTTP 402 responses and negotiates payment terms
- **Base wallet integration** — native support for EVM wallets on Base
- **Configurable spending policies** — per-tx limits, daily caps, merchant allowlists
- **Transaction receipts** — full audit trail for every payment
- **Dispute resolution** — built-in hooks for on-chain dispute handling

---

## API Reference

### `MoltMCPClient`

```ts
const client = new MoltMCPClient(config: MoltMCPConfig)
```

#### Config

| Field | Type | Description |
|-------|------|-------------|
| `baseRpc` | `string` | Base RPC URL |
| `wallet` | `Wallet` | EVM wallet instance |
| `policies` | `SpendingPolicy` | Optional spending constraints |

#### Methods

| Method | Description |
|--------|-------------|
| `payForResource(params)` | Automatically pay for an HTTP 402 resource |
| `executePayment(params)` | Execute a direct payment to a merchant |
| `verifyPayment(credential)` | Verify an incoming payment credential |
| `getTransactionHistory()` | Retrieve past transactions |

---

## Examples

### Basic Payment

```ts
const payment = await client.executePayment({
  amount: 5.00,
  merchant: '0xMerchantAddress',
  token: 'USDC',
  memo: 'payment-id-12345'
})

console.log(`Tx hash: ${payment.hash}`)
```

### Verify Incoming Payment (Merchant Side)

```ts
import { verifyPayment } from '@moltmcp/sdk'

app.get('/api/premium', async (req, res) => {
  const credential = req.headers['mpp-payment-credential']

  if (!credential) {
    return res.status(402).json({
      amount: '10.00',
      currency: 'USD',
      payment_methods: [{
        type: 'crypto',
        blockchain: 'base',
        token: 'USDC',
        recipient: process.env.MERCHANT_WALLET
      }]
    })
  }

  const payment = await verifyPayment(credential)
  if (payment.verified) return res.json({ data: getPremiumData() })
  return res.status(402).json({ error: 'Invalid payment' })
})
```

---

## Development

```bash
git clone https://github.com/jackyixuan/moltmcp-sdk.git
cd moltmcp-sdk
npm install
npm run build
npm test
```

---

## License

[MIT](LICENSE) © 2026 MoltMCP
