# 📚 OrbitStream Documentation

> Real-time token streaming on Stellar — official technical documentation.

[![Stellar](https://img.shields.io/badge/Stellar-Soroban-7C68EE)](https://stellar.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## What is OrbitStream?

OrbitStream is a DeFi protocol built on Stellar that enables **continuous, per-second token streaming**. Instead of sending lump-sum payments, senders lock tokens into a Soroban smart contract and recipients accrue them every second — claimable at any time.

```
Sender locks 1,000 XLM → contract streams 0.0347 XLM/sec → Recipient claims anytime
```

---

## Quick Links

| Resource | Description |
|----------|-------------|
| [Getting Started](./docs/getting-started.md) | Set up and run OrbitStream locally |
| [Smart Contracts](./docs/contracts.md) | Contract API, types, and error codes |
| [REST API](./docs/api.md) | Full backend API reference |
| [WebSocket Events](./docs/websocket.md) | Real-time event subscription |
| [Integration Guide](./docs/integration.md) | SDK usage and code examples |
| [Deployment](./docs/deployment.md) | Deploy contracts, backend, and frontend |
| [Contributing](./docs/contributing.md) | How to contribute to OrbitStream |

---

## Protocol Overview

```
┌─────────────────────────────────────────────────┐
│           OrbitStream Frontend (Next.js)         │
│   Freighter wallet · Stream dashboard · Live UI  │
└──────────────┬──────────────────┬───────────────┘
               │ REST             │ WebSocket
┌──────────────▼──────────────────▼───────────────┐
│           OrbitStream Backend (NestJS)           │
│   JWT auth · Stream API · Stellar Horizon · WS   │
└──────────────────────┬──────────────────────────┘
                       │ Soroban RPC
┌──────────────────────▼──────────────────────────┐
│      OrbitStream Contract (Rust/Soroban)         │
│   create · claim · pause · resume · cancel       │
└─────────────────────────────────────────────────┘
```

---

## Repositories

| Repo | Purpose | Language |
|------|---------|----------|
| [orbitstream-contracts](https://github.com/OrbitStream/orbitstream-contracts) | Soroban smart contracts | Rust |
| [OrbitStream_backend](https://github.com/OrbitStream/OrbitStream_backend) | REST API + WebSocket | TypeScript/NestJS |
| [orbitstream-frontend](https://github.com/OrbitStream/orbitstream-frontend) | Web dashboard | TypeScript/Next.js |
| [orbitstream-docs](https://github.com/OrbitStream/orbitstream-docs) | This documentation | Markdown |

---

## Stream Lifecycle

```
                    ┌──────────┐
                    │  Active  │◄──────────────────┐
                    └────┬─────┘                   │
              pause()    │          resume()        │
                    ┌────▼─────┐                   │
                    │  Paused  │───────────────────►│
                    └────┬─────┘
              cancel()   │    cancel() / admin_cancel()
                    ┌────▼──────────────────────────┐
                    │          Cancelled             │
                    │  recipient ← earned tokens     │
                    │  sender   ← unearned refund    │
                    └───────────────────────────────┘
              (end_time reached + fully claimed)
                    ┌──────────────┐
                    │  Completed   │
                    └──────────────┘
```

---

## Use Cases

| Use Case | Description |
|----------|-------------|
| 💼 **Payroll** | Pay employees per second. No more month-end payroll runs. |
| 📱 **Subscriptions** | Charge per second of usage. Users pay exactly what they consume. |
| 🎓 **Grants** | Stream funding to builders. Automatic, milestone-free disbursement. |
| 🤝 **Vesting** | Stream token vesting to team members. Transparent and unstoppable. |
| 🎮 **Gaming** | Reward players per second of active play. |

---

## License

MIT License. Copyright (c) 2026 OrbitStream Protocol.
