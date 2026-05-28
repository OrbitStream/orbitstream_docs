# Stellar Checkout — Architecture Overview

## System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Stellar Checkout                          │
├──────────────┬──────────────┬──────────────┬────────────────┤
│  Frontend    │  Backend     │  Indexer     │  Dashboard     │
│              │              │              │                │
│  - React SDK │  - Payment   │  - Stellar   │  - Merchant    │
│  - Checkout  │    detection │    ledger    │    portal      │
│    page      │  - Webhook   │    watcher   │  - Analytics   │
│  - Widget    │    dispatch  │  - Event     │  - Settings    │
│  - QR codes  │  - Session   │    indexing  │  - Webhooks    │
│              │    mgmt      │              │    config      │
├──────────────┴──────────────┴──────────────┴────────────────┤
│                    Stellar Network                           │
│  - Stellar SDK (JS/Python)                                  │
│  - SEP-24 (fiat on/off ramps)                               │
│  - Stellar Asset Contract (Soroban)                         │
│  - Built-in DEX (multi-asset)                               │
│  - Anchor Network (fiat settlement)                         │
└─────────────────────────────────────────────────────────────┘
```

## Components

### Backend (NestJS)
- **Merchant API** — registration, API key management, webhook configuration
- **Checkout API** — session creation, payment URL generation, status polling
- **Payment Detector** — Horizon streaming, memo-based payment matching
- **Webhook Dispatcher** — HMAC-signed event delivery with retry

### Frontend (Next.js)
- **Checkout Page** — customer-facing payment page with QR code and wallet connect
- **Merchant Dashboard** — API key management, transaction history, webhook config

### Smart Contract (Soroban)
- **Escrow Contract** — holds funds for dispute-prone transactions with timeout-based refund

### JS SDK
- **@stellar-checkout/sdk** — drop-in integration for creating checkout sessions

## Data Flow

1. Merchant creates checkout session via API (authenticated with API key)
2. Backend generates unique memo, stores session, returns checkout URL
3. Customer visits checkout page, sees amount + QR code
4. Customer pays via wallet or QR code scan
5. Payment detector matches incoming payment by memo
6. Backend marks session as paid, records payment, dispatches webhook
7. Customer sees confirmation screen

## Payment Detection

Uses Horizon streaming (`/accounts/{id}/payments`) with cursor-based pagination. Redis persists the last processed cursor for restart recovery. Each checkout session has a unique hex memo for matching.
