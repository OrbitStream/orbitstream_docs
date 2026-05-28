# Merchant Setup Guide

## Getting Started

### 1. Register

Connect your Stellar wallet and register your business.

```bash
curl -X POST http://localhost:3001/merchants/register \
  -H "Content-Type: application/json" \
  -d '{
    "walletAddress": "GABC...1234",
    "businessName": "Acme Corp",
    "email": "merchant@example.com"
  }'
```

### 2. Login

Sign a challenge with your wallet to get a JWT token. OrbitStream uses SEP-10 for wallet-based authentication — the login endpoint returns a challenge transaction that you sign with your Stellar wallet.

```bash
curl -X POST http://localhost:3001/auth/login \
  -H "Content-Type: application/json" \
  -d '{"walletAddress": "GABC...1234"}'
```

### 3. Generate API Key

```bash
curl -X POST http://localhost:3001/merchants/me/api-keys \
  -H "Authorization: Bearer <jwt>" \
  -H "Content-Type: application/json" \
  -d '{"environment": "testnet"}'
```

**Save the key immediately** — it's only shown once.

### 4. Configure Webhook (Optional)

```bash
curl -X PATCH http://localhost:3001/merchants/me/webhook \
  -H "Authorization: Bearer <jwt>" \
  -H "Content-Type: application/json" \
  -d '{"webhookUrl": "https://your-server.com/webhooks"}'
```

Save the `webhookSecret` — use it to verify webhook signatures.

### 5. Create a Checkout Session

```bash
curl -X POST http://localhost:3001/v1/checkout/sessions \
  -H "Authorization: Bearer sk_test_..." \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 25.00,
    "asset": "USDC",
    "successUrl": "https://example.com/success",
    "cancelUrl": "https://example.com/cancel"
  }'
```

Response includes a `url` — redirect your customer there.

## API Key Environments

| Prefix | Environment | Use |
|--------|-------------|-----|
| `sk_test_` | Testnet | Development and testing |
| `sk_live_` | Mainnet | Production |

## Fiat Settlement (SEP-24)

Merchants can withdraw their USDC balance to local fiat currency via Stellar anchors.

### Connect an Anchor

```bash
curl -X PATCH http://localhost:3001/merchants/me/anchor \
  -H "Authorization: Bearer <JWT>" \
  -H "Content-Type: application/json" \
  -d '{
    "anchorDomain": "anchor.example.com"
  }'
```

OrbitStream fetches the anchor's TOML file, validates SEP compliance, and initiates SEP-24 deposit/withdrawal flows.

### Withdraw to Fiat

Once an anchor is connected, merchants can request fiat withdrawal through the dashboard or API. The anchor handles KYC (SEP-12) and processes the fiat transfer to the merchant's bank account.

**Supported settlement currencies** depend on the connected anchor. Common anchors support USD, EUR, ARS, BRL, NGN, and others.

See [Stellar Features](../architecture/stellar-features.md) for details on SEP protocol integration.
