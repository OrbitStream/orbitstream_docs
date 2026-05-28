# Backend Architecture

## Stack
- NestJS 10 (TypeScript)
- Drizzle ORM + PostgreSQL
- Redis (cursor persistence)
- Stellar SDK

## Modules

### AuthModule
- SEP-10 wallet-based login for merchant dashboard (signs a challenge transaction to prove account ownership)
- API key guard for programmatic access
- JwtStrategy validates JWT, returns walletAddress

### MerchantsModule
- Registration (wallet address + business info)
- API key CRUD (generate, list, revoke)
- Webhook URL configuration
- Anchor connection for fiat settlement (validates anchor TOML, stores anchor domain)

### CheckoutModule
- Session creation with unique memo or muxed account
- Multi-asset support: `acceptAssets` parameter specifies which assets the customer can pay in
- DEX price lookup for real-time conversion rates across accepted assets
- Session status (public endpoint for customer polling)
- Session cancellation

### PaymentsModule
- PaymentDetectorService — polls Horizon for incoming payments
- **Memo-based matching** (default): matches payments via 16-char hex memo
- **Muxed account matching** (optional): matches payments via unique M-account destination
- Triggers webhook dispatch on confirmation

### WebhookModule
- HMAC-SHA256 signed delivery
- Delivery tracking in webhook_deliveries table
- Exponential backoff retry (max 5 attempts)

### StellarModule
- Horizon account info and balance queries
- Transaction verification
- Payment history polling
- DEX order book queries for multi-asset price conversion
- Claimable Balance creation and management for simple escrow flows
- SEP-12 KYC data submission to anchors
- SEP-24 deposit/withdrawal initiation for fiat settlement

### AnchorsModule (Phase 2)
- Anchor TOML discovery and validation
- SEP-24 deposit/withdrawal flow orchestration
- SEP-31 cross-border payment routing
- Fiat settlement status tracking

### MonitoringModule
- Health checks (Terminus)
- Prometheus metrics

## Database Schema

### merchants
id, wallet_address (unique), business_name, email, webhook_url, webhook_secret, logo_url, anchor_domain, created_at

### api_keys
id, merchant_id (FK), key_prefix, key_hash, environment (testnet/mainnet), is_active, created_at

### checkout_sessions
id, merchant_id (FK), amount, asset_code, asset_issuer, accept_assets (jsonb), receiving_account, muxed_account, memo, status, success_url, cancel_url, metadata (jsonb), expires_at, created_at

### payments
id, session_id (FK), merchant_id (FK), tx_hash (unique), amount, asset_code, asset_issuer, sender_address, confirmed_at, created_at

### webhook_deliveries
id, merchant_id (FK), event, payload (jsonb), response_status, delivered_at, attempts, next_retry_at, created_at

### claimable_escrows
id, session_id (FK), merchant_id (FK), buyer_address, seller_address, asset_code, asset_issuer, amount, balance_id (unique), timeout_at, status (active/claimed/expired), created_at

### anchors
id, merchant_id (FK), anchor_domain, toml_url, supported_currencies (jsonb), is_active, created_at

## API Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /merchants/register | None | Register merchant |
| POST | /auth/login | None | SEP-10 wallet login |
| GET | /merchants/me | JWT | Get profile |
| PATCH | /merchants/me | JWT | Update profile |
| POST | /merchants/me/api-keys | JWT | Generate API key |
| GET | /merchants/me/api-keys | JWT | List keys |
| DELETE | /merchants/me/api-keys/:id | JWT | Revoke key |
| PATCH | /merchants/me/webhook | JWT | Set webhook URL |
| PATCH | /merchants/me/anchor | JWT | Connect anchor for fiat settlement |
| POST | /v1/checkout/sessions | API Key | Create session |
| GET | /v1/checkout/sessions/:id | None | Get session status |
| POST | /v1/checkout/sessions/:id/cancel | API Key | Cancel session |
| POST | /v1/escrow/claimable | API Key | Create claimable balance escrow |
| POST | /v1/escrow/claimable/:id/claim | API Key | Claim a claimable balance |
| GET | /v1/escrow/claimable/:id | None | Get claimable escrow status |
| GET | /health | None | Health check |
| GET | /metrics | None | Prometheus metrics |
