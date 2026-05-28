# OrbitStream — Architecture Overview

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         OrbitStream                                  │
├──────────────┬──────────────┬──────────────┬────────────────────────┤
│  Frontend    │  Backend     │  Indexer     │  Dashboard             │
│              │              │              │                        │
│  - React SDK │  - Payment   │  - Stellar   │  - Merchant portal     │
│  - Checkout  │    detection │    ledger    │  - Analytics           │
│    page      │  - Webhook   │    watcher   │  - Settings            │
│  - Widget    │    dispatch  │  - Event     │  - Webhook config      │
│    (Web      │  - Session   │    indexing   │  - Anchor management   │
│    Components│    mgmt      │              │  - Fiat settlement     │
│  - QR codes  │  - SEP auth  │              │                        │
│              │  - DEX prices│              │                        │
├──────────────┴──────────────┴──────────────┴────────────────────────┤
│                        Stellar Network                               │
│  - Stellar SDK (JS/Python)                                          │
│  - SEP-10 (authentication)                                          │
│  - SEP-12 (KYC)                                                     │
│  - SEP-24 (fiat on/off ramps)                                       │
│  - SEP-31 (cross-border payments)                                   │
│  - Built-in DEX (multi-asset conversion)                            │
│  - Muxed Accounts (sub-addressing)                                  │
│  - Claimable Balances (escrow without contracts)                    │
│  - Anchor Network (fiat settlement)                                 │
└─────────────────────────────────────────────────────────────────────┘
```

## Components

### Backend (NestJS)
- **Merchant API** — registration, API key management, webhook configuration, anchor connection
- **Checkout API** — session creation, payment URL generation, status polling, multi-asset support
- **Payment Detector** — Horizon streaming, memo-based or muxed-account payment matching
- **DEX Price Service** — real-time order book queries for multi-asset conversion rates
- **Webhook Dispatcher** — HMAC-signed event delivery with retry

### Frontend (Next.js + Web Components)
- **Checkout Page** — customer-facing payment page with QR code, wallet connect, and multi-asset selector
- **Embeddable Widget** — framework-agnostic Web Component (`<orbitstream-checkout>`) for any page
- **Merchant Dashboard** — API key management, transaction history, webhook config, fiat settlement

### Escrow Layer
- **Claimable Balances** (Stellar Classic) — simple escrow for marketplace transactions, no contract needed
- **Soroban Escrow Contract** — programmable escrow for complex flows with custom logic or dispute resolution

### JS SDK
- **@orbitstream/sdk** — drop-in integration for creating checkout sessions
- **@orbitstream/sdk/react** — React component wrapper for the embeddable widget

## Stellar-Native Features

OrbitStream uses Stellar's built-in primitives rather than reinventing them:

| Feature | How OrbitStream Uses It |
|---------|------------------------|
| **SEP-10** | Wallet-based authentication — merchants prove account ownership |
| **SEP-12** | KYC collection for anchor fiat settlement compliance |
| **SEP-24** | Hosted fiat deposit/withdrawal via anchor iframe |
| **SEP-31** | Cross-border merchant settlement |
| **Muxed Accounts** | Sub-addressing for payment matching without memos |
| **Claimable Balances** | Escrow without smart contracts |
| **Built-in DEX** | Multi-asset acceptance with auto-conversion |

See [Stellar Features](stellar-features.md) for detailed documentation.

## Data Flow

1. Merchant creates checkout session via API (authenticated with API key)
2. Backend generates unique memo (or muxed account), stores session, returns checkout URL
3. Customer visits checkout page, sees amount + asset options with DEX prices
4. Customer selects asset, pays via wallet or QR code scan
5. Payment detector matches incoming payment by memo or muxed account
6. Backend marks session as paid, records payment, dispatches webhook
7. Customer sees confirmation screen

## Payment Detection

Two modes for matching incoming payments to checkout sessions:

**Memo-based (default):**
Uses Horizon streaming (`/accounts/{id}/payments`) with cursor-based pagination. Redis persists the last processed cursor for restart recovery. Each checkout session has a unique 16-char hex memo.

**Muxed accounts (optional):**
Each session gets a unique muxed account (M-account) derived from the merchant's Stellar address. The destination address itself identifies the session — no memo needed. Better UX for wallets that don't support custom memos.
