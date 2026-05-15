# OrbitStream Documentation

> Real-time token streaming on Stellar — technical reference and integration guides.

---

## Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Smart Contract Reference](#smart-contract-reference)
- [Backend API Reference](#backend-api-reference)
- [WebSocket Events](#websocket-events)
- [Integration Guide](#integration-guide)
- [Deployment](#deployment)

---

## Overview

OrbitStream is a DeFi protocol that enables continuous, per-second token streaming on Stellar. Instead of sending a lump-sum payment, senders lock tokens into a Soroban smart contract and the recipient accrues them every second — claimable at any time.

**Core components:**

| Repo | Purpose |
|------|---------|
| `orbitstream-contracts` | Soroban smart contracts (stream logic) |
| `OrbitStream_backend` | NestJS REST API + WebSocket gateway |
| `orbitstream-frontend` | Next.js web dashboard |
| `orbitstream-docs` | This documentation |

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│           OrbitStream Frontend (Next.js)         │
│   • Freighter wallet integration                 │
│   • Real-time stream dashboard                   │
│   • WebSocket live updates                       │
└──────────────┬──────────────────┬───────────────┘
               │ REST API         │ WebSocket
┌──────────────▼──────────────────▼───────────────┐
│           OrbitStream Backend (NestJS)           │
│   • JWT auth (wallet-signed)                     │
│   • Stream state management                      │
│   • Stellar Horizon integration                  │
│   • Prometheus metrics                           │
└──────────────────────┬──────────────────────────┘
                       │ Soroban RPC
┌──────────────────────▼──────────────────────────┐
│      OrbitStream Contract (Soroban/Rust)         │
│   • create_stream  • claim                       │
│   • pause_stream   • resume_stream               │
│   • cancel_stream  • top_up                      │
└─────────────────────────────────────────────────┘
```

---

## Smart Contract Reference

**Contract:** `orbitstream-stream`  
**Network:** Stellar Testnet / Mainnet  
**Language:** Rust (Soroban SDK)

### Data Types

#### `StreamStatus`
```rust
enum StreamStatus { Active, Paused, Cancelled, Completed }
```

#### `Stream`
```rust
struct Stream {
    id:                   u64,
    sender:               Address,
    recipient:            Address,
    token:                Address,     // Stellar asset contract
    rate_per_second:      i128,        // token units per active second
    start_time:           u64,         // Unix timestamp
    end_time:             u64,         // 0 = open-ended
    total_deposited:      i128,
    total_claimed:        i128,
    status:               StreamStatus,
    paused_at:            u64,
    total_paused_seconds: u64,
}
```

### Functions

#### `initialize(env, admin)`
One-time setup. Sets the admin address.

#### `create_stream(env, sender, recipient, token, rate_per_second, duration_seconds, deposit) → u64`
Creates a stream and locks `deposit` tokens. Returns the stream ID.

| Parameter | Type | Description |
|-----------|------|-------------|
| `sender` | Address | Stream funder (must sign) |
| `recipient` | Address | Receiving address |
| `token` | Address | Stellar asset contract |
| `rate_per_second` | i128 | Token units per active second |
| `duration_seconds` | u64 | Stream length; 0 = open-ended |
| `deposit` | i128 | Tokens locked upfront (≥ 1s of rate) |

#### `top_up(env, sender, stream_id, amount)`
Adds more tokens to an existing stream. Extends runway.

#### `claim(env, recipient, stream_id) → i128`
Recipient claims all accrued tokens. Returns amount transferred.

#### `pause_stream(env, sender, stream_id)`
Freezes the clock. No tokens accrue while paused.

#### `resume_stream(env, sender, stream_id)`
Resumes the stream. Paused duration is excluded from earnings.

#### `cancel_stream(env, sender, stream_id)`
Cancels the stream. Recipient receives earned tokens; sender receives unearned remainder.

#### `get_stream(env, stream_id) → Option<Stream>`
Read-only. Returns the full stream struct.

#### `claimable_amount(env, stream_id) → i128`
Read-only. Returns tokens the recipient can claim right now.

#### `stream_count(env) → u64`
Total number of streams created.

### Error Codes

| Code | Name | Meaning |
|------|------|---------|
| 1 | AlreadyInitialized | Contract already set up |
| 2 | NotInitialized | Contract not set up |
| 3 | Unauthorized | Caller is not the stream owner |
| 4 | NotFound | Stream ID does not exist |
| 5 | InvalidRate | Rate must be > 0 |
| 6 | InvalidDeposit | Deposit must be > 0 |
| 7 | SelfStream | Sender and recipient cannot be the same |
| 8 | StreamNotActive | Stream must be active for this operation |
| 9 | StreamNotPaused | Stream must be paused to resume |
| 10 | NothingToClaim | No tokens have accrued |
| 11 | AlreadyTerminated | Stream is cancelled or completed |
| 12 | InsufficientFunds | Deposit does not cover one second |

---

## Backend API Reference

Base URL: `https://your-deployment/api/v1`

All endpoints except `/auth/login` require a `Bearer` token in the `Authorization` header.

### Authentication

#### `POST /auth/login`
```json
// Request
{ "walletAddress": "GAXYZ...3A4B" }

// Response
{ "access_token": "eyJ...", "wallet": "GAXYZ...3A4B" }
```

### Streams

#### `POST /streams` — Create stream
```json
// Request
{
  "recipient": "GBCD...7E2F",
  "tokenAddress": "CBIELTK6YBZJU5UP2WWQEUCYKLPU6AUNZ2BQ4WWFEIE3USCIHMXQDAMA",
  "ratePerSecond": 347222,
  "totalDeposited": 1000000000,
  "durationSeconds": 2592000
}
```

#### `GET /streams` — List my streams
Query params: `?status=active&role=sender`

#### `GET /streams/:id` — Get stream

#### `GET /streams/:id/claimable` — Claimable now
```json
{ "streamId": "uuid", "claimable": 1234567, "earned": 9876543 }
```

#### `PATCH /streams/:id/claim` — Record claim
```json
{ "txHash": "abc123..." }
```

#### `PATCH /streams/:id/pause` — Pause stream

#### `PATCH /streams/:id/resume` — Resume stream

#### `DELETE /streams/:id` — Cancel stream

### Monitoring

| Endpoint | Description |
|----------|-------------|
| `GET /health` | Liveness probe |
| `GET /metrics` | Prometheus metrics |

---

## WebSocket Events

Connect to `wss://your-deployment`.

### Subscribe to a stream
```js
socket.emit('subscribe_stream', 'stream-uuid');
```

### Incoming events
```js
socket.on('stream_update',  ({ streamId, ...data }) => { /* state change */ });
socket.on('claimed',        ({ streamId, amount, recipient }) => { /* claim */ });
socket.on('status_change',  ({ streamId, status }) => { /* pause/resume/cancel */ });
```

---

## Integration Guide

### 1. Create a stream via SDK

```typescript
import { Contract, xdr, TransactionBuilder, Networks } from '@stellar/stellar-sdk';

const contract = new Contract(STREAM_CONTRACT_ID);
const tx = new TransactionBuilder(sourceAccount, { fee: '100' })
  .addOperation(contract.call(
    'create_stream',
    xdr.ScVal.scvAddress(senderAddress),
    xdr.ScVal.scvAddress(recipientAddress),
    xdr.ScVal.scvAddress(tokenContractId),
    xdr.ScVal.scvI128(new xdr.Int128Parts({ lo: 347222n, hi: 0n })), // rate
    xdr.ScVal.scvU64(xdr.Uint64.fromString('2592000')),               // 30 days
    xdr.ScVal.scvI128(new xdr.Int128Parts({ lo: 1000000000n, hi: 0n })), // deposit
  ))
  .setNetworkPassphrase(Networks.TESTNET)
  .setTimeout(30)
  .build();
```

### 2. Listen to live updates
```typescript
import { io } from 'socket.io-client';

const socket = io('wss://api.orbitstream.xyz');
socket.emit('subscribe_stream', streamId);
socket.on('claimed', ({ amount }) => {
  console.log(`${amount} tokens claimed`);
});
```

---

## Deployment

### Contracts
```bash
cd orbitstream-contracts/Contract
cargo build --target wasm32-unknown-unknown --release
stellar contract deploy --wasm target/wasm32-unknown-unknown/release/orbitstream_stream.wasm --network testnet
```

### Backend
```bash
cd OrbitStream_backend
cp .env.example .env   # fill in contract IDs
npm install
npm run start:prod
```

### Frontend
```bash
cd orbitstream-frontend
npm install
npm run build
```

---

*© 2026 OrbitStream Protocol · MIT License*
