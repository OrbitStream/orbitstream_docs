# Getting Started

This guide walks you through running OrbitStream locally end-to-end — contracts, backend, and frontend.

---

## Prerequisites

| Tool | Version | Install |
|------|---------|---------|
| Rust | 1.74+ | [rustup.rs](https://rustup.rs) |
| Node.js | 20+ | [nodejs.org](https://nodejs.org) |
| Stellar CLI | latest | `cargo install stellar-cli --features opt` |
| Freighter | latest | [freighter.app](https://freighter.app) |
| PostgreSQL | 14+ | [postgresql.org](https://postgresql.org) |

```bash
# Add Soroban target
rustup target add wasm32-unknown-unknown
```

---

## 1. Clone all repositories

```bash
git clone https://github.com/OrbitStream/orbitstream-contracts
git clone https://github.com/OrbitStream/OrbitStream_backend
git clone https://github.com/OrbitStream/orbitstream-frontend
```

---

## 2. Deploy the contract

```bash
cd orbitstream-contracts/Contract

# Build
cargo build --target wasm32-unknown-unknown --release

# Deploy to testnet (returns a contract ID)
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/orbitstream_stream.wasm \
  --source <your-secret-key> \
  --network testnet

# Initialize (replace CONTRACT_ID and ADMIN_ADDRESS)
stellar contract invoke \
  --id <CONTRACT_ID> \
  --source <your-secret-key> \
  --network testnet \
  -- initialize --admin <ADMIN_ADDRESS>
```

Save the `CONTRACT_ID` — you'll need it for the backend and frontend.

---

## 3. Start the backend

```bash
cd OrbitStream_backend
npm install
cp .env.example .env
```

Edit `.env`:
```env
DATABASE_URL=postgresql://postgres:password@localhost:5432/orbitstream
JWT_SECRET=your-long-random-secret
STELLAR_HORIZON_URL=https://horizon-testnet.stellar.org
STELLAR_RPC_URL=https://soroban-testnet.stellar.org
STREAM_CONTRACT_ID=<YOUR_CONTRACT_ID>
```

```bash
npm run start:dev
# API running at http://localhost:3001/api/v1
```

---

## 4. Start the frontend

```bash
cd orbitstream-frontend
npm install
```

Create `.env.local`:
```env
NEXT_PUBLIC_STELLAR_NETWORK=TESTNET
NEXT_PUBLIC_BACKEND_URL=http://localhost:3001
NEXT_PUBLIC_WS_URL=ws://localhost:3001
NEXT_PUBLIC_STREAM_CONTRACT_ID=<YOUR_CONTRACT_ID>
```

```bash
npm run dev
# App running at http://localhost:3000
```

---

## 5. Test the full flow

1. Open [http://localhost:3000](http://localhost:3000)
2. Click **Connect Wallet** — approve in Freighter
3. Click **Create Stream** — set a recipient, rate, and deposit
4. Watch tokens accrue in real time on the dashboard
5. Click **Claim** to withdraw earned tokens

---

## Next Steps

- [Smart Contract Reference](./contracts.md)
- [REST API Reference](./api.md)
- [Integration Guide](./integration.md)
