# Deployment Guide

---

## 1. Deploy the Soroban contract

### Build

```bash
cd orbitstream-contracts/Contract
cargo build --target wasm32-unknown-unknown --release
```

The compiled WASM will be at:
```
target/wasm32-unknown-unknown/release/orbitstream_stream.wasm
```

### Deploy to Testnet

```bash
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/orbitstream_stream.wasm \
  --source <DEPLOYER_SECRET_KEY> \
  --network testnet
```

**Output:** `CONTRACT_ID` — save this.

### Initialize the contract

```bash
stellar contract invoke \
  --id <CONTRACT_ID> \
  --source <ADMIN_SECRET_KEY> \
  --network testnet \
  -- initialize \
  --admin <ADMIN_STELLAR_ADDRESS>
```

### Deploy to Mainnet

Replace `--network testnet` with `--network mainnet` and ensure you have sufficient XLM for fees.

---

## 2. Deploy the backend

### Environment variables

```env
PORT=3001
NODE_ENV=production
DATABASE_URL=postgresql://user:pass@host:5432/orbitstream
JWT_SECRET=<long-random-string-min-32-chars>
STELLAR_NETWORK=MAINNET
STELLAR_HORIZON_URL=https://horizon.stellar.org
STELLAR_RPC_URL=https://soroban-rpc.stellar.org
STREAM_CONTRACT_ID=<YOUR_CONTRACT_ID>
WS_CORS_ORIGIN=https://app.orbitstream.xyz
```

### Docker (recommended)

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY dist ./dist
EXPOSE 3001
CMD ["node", "dist/main"]
```

```bash
npm run build
docker build -t orbitstream-backend .
docker run -p 3001:3001 --env-file .env orbitstream-backend
```

### Bare metal

```bash
npm install
npm run build
npm run start:prod
```

### Database setup

TypeORM auto-creates tables in `development` mode. For production, run migrations:

```bash
npm run migration:run
```

---

## 3. Deploy the frontend

### Vercel (recommended)

```bash
npm install -g vercel
cd orbitstream-frontend
vercel --prod
```

Set environment variables in the Vercel dashboard:
```
NEXT_PUBLIC_STELLAR_NETWORK=MAINNET
NEXT_PUBLIC_BACKEND_URL=https://api.orbitstream.xyz
NEXT_PUBLIC_WS_URL=wss://api.orbitstream.xyz
NEXT_PUBLIC_STREAM_CONTRACT_ID=<YOUR_CONTRACT_ID>
```

### Netlify

```toml
# netlify.toml
[build]
  command = "npm run build"
  publish = ".next"

[[plugins]]
  package = "@netlify/plugin-nextjs"
```

### Self-hosted

```bash
npm run build
npm start  # runs on port 3000
```

---

## 4. Health checks

After deployment, verify everything is running:

```bash
# Backend health
curl https://api.orbitstream.xyz/api/v1/health

# Metrics
curl https://api.orbitstream.xyz/api/v1/metrics

# Contract (read a stream count)
stellar contract invoke \
  --id <CONTRACT_ID> \
  --network mainnet \
  -- stream_count
```

---

## Infrastructure checklist

- [ ] Contract deployed and initialized
- [ ] PostgreSQL database provisioned
- [ ] Backend running and connected to DB
- [ ] `JWT_SECRET` set to a secure random value
- [ ] `STREAM_CONTRACT_ID` set in backend and frontend
- [ ] CORS configured (`WS_CORS_ORIGIN`)
- [ ] SSL/TLS on backend (required for `wss://`)
- [ ] Frontend deployed and env vars set
- [ ] Health check returning `{ "status": "ok" }`
