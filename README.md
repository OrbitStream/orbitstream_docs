# OrbitStream Documentation

[![Stellar](https://img.shields.io/badge/Stellar-Soroban-7C68EE)](https://stellar.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> **Technical documentation for OrbitStream — a Stripe-like merchant payment gateway for Stellar.**

OrbitStream provides the missing merchant layer for Stellar: a hosted checkout page, embeddable widget, JS SDK, and webhook system — so any merchant can start accepting Stellar payments in under 10 minutes.

---

## Quick Links

| Resource | Description |
|----------|-------------|
| [Architecture Overview](./architecture/overview.md) | System design and data flow |
| [Backend Design](./architecture/backend.md) | API endpoints, modules, database schema |
| [Frontend Design](./architecture/frontend.md) | Pages, components, hooks, widget |
| [Stellar Features](./architecture/stellar-features.md) | SEP protocols, DEX, muxed accounts, claimable balances |
| [Escrow Architecture](./architecture/contract.md) | Claimable Balances + Soroban escrow |
| [Contract Spec](./contract/spec.md) | Escrow contract functions and types |
| [API Reference](./api/openapi.yaml) | OpenAPI specification |
| [Integration Guide](./guides/integration-guide.md) | SDK usage, widget, webhooks |
| [Merchant Setup](./guides/merchant-setup.md) | Registration, API keys, fiat settlement |
| [Roadmap](./roadmap.md) | MVP phases and timeline |
| [Competitive Landscape](./competitive.md) | How OrbitStream compares |
| [Security](./security/threat-model.md) | Threat model and mitigations |

---

## How It Works

```
Customer                       OrbitStream                    Merchant
--------                       ---------                    --------
   |                                |                          |
   |  Click "Pay"                   |                          |
   |------------------------------->|                          |
   |                                |                          |
   |  Checkout page loads           |                          |
   |  (shows amount + asset options)|                          |
   |<-------------------------------|                          |
   |                                |                          |
   |  Select asset, scan QR         |                          |
   |  or connect wallet             |                          |
   |------------------------------->|                          |
   |                                |                          |
   |  Sign payment transaction      |                          |
   |------------------------------->|                          |
   |                                |  Detect payment on       |
   |                                |  Stellar ledger          |
   |                                |  (memo or muxed account) |
   |                                |                          |
   |                                |  Webhook: payment.confirmed
   |                                |------------------------->|
   |                                |                          |
   |  Confirmation screen           |                          |
   |<-------------------------------|                          |
```

---

## Stellar-Native Features

OrbitStream is built on Stellar's existing primitives:

| Feature | Description |
|---------|-------------|
| **SEP-10** | Wallet-based authentication |
| **SEP-24** | Fiat settlement via anchor iframe |
| **Built-in DEX** | Multi-asset acceptance with auto-conversion |
| **Muxed Accounts** | Payment matching without memos |
| **Claimable Balances** | Escrow without smart contracts |

See [Stellar Features](./architecture/stellar-features.md) for details.

---

## Repositories

| Repo | Purpose | Language |
|------|---------|----------|
| [orbitstream_backend](https://github.com/OrbitStream/orbitstream_backend) | REST API + payment detection | TypeScript/NestJS |
| [orbitstream_contracts](https://github.com/OrbitStream/orbitstream_contracts) | Escrow smart contract | Rust/Soroban |
| [orbitstream_frontend](https://github.com/OrbitStream/orbitstream_frontend) | Checkout UI + dashboard | TypeScript/Next.js |
| [@orbitstream/sdk](https://github.com/OrbitStream/orbitstream-sdk) | JS/TS integration SDK | TypeScript |
| orbitstream_docs | This documentation | Markdown |

---

## License

MIT License. Copyright (c) 2026 OrbitStream.
