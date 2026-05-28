# Backend Architecture

## Stack
- NestJS 10 (TypeScript)
- Drizzle ORM + PostgreSQL
- Redis (cursor persistence)
- Stellar SDK

## Modules

### AuthModule
- JWT wallet-based login for merchant dashboard
- API key guard for programmatic access
- JwtStrategy validates JWT, returns walletAddress

### MerchantsModule
- Registration (wallet address + business info)
- API key CRUD (generate, list, revoke)
- Webhook URL configuration

### CheckoutModule
- Session creation with unique memo
- Session status (public endpoint for customer polling)
- Session cancellation

### PaymentsModule
- PaymentDetectorService — polls Horizon for incoming payments
- Memo-based matching to pending sessions
- Triggers webhook dispatch on confirmation

### WebhookModule
- HMAC-SHA256 signed delivery
- Delivery tracking in webhook_deliveries table
- Exponential backoff retry (max 5 attempts)

### StellarModule
- Horizon account info and balance queries
- Transaction verification
- Payment history polling

### MonitoringModule
- Health checks (Terminus)
- Prometheus metrics

## Database Schema

### merchants
id, wallet_address (unique), business_name, email, webhook_url, webhook_secret, logo_url, created_at

### api_keys
id, merchant_id (FK), key_prefix, key_hash, environment (testnet/mainnet), is_active, created_at

### checkout_sessions
id, merchant_id (FK), amount, asset_code, asset_issuer, receiving_account, memo, status, success_url, cancel_url, metadata (jsonb), expires_at, created_at

### payments
id, session_id (FK), merchant_id (FK), tx_hash (unique), amount, asset_code, asset_issuer, sender_address, confirmed_at, created_at

### webhook_deliveries
id, merchant_id (FK), event, payload (jsonb), response_status, delivered_at, attempts, next_retry_at, created_at

## API Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /merchants/register | None | Register merchant |
| POST | /auth/login | None | Wallet login |
| GET | /merchants/me | JWT | Get profile |
| PATCH | /merchants/me | JWT | Update profile |
| POST | /merchants/me/api-keys | JWT | Generate API key |
| GET | /merchants/me/api-keys | JWT | List keys |
| DELETE | /merchants/me/api-keys/:id | JWT | Revoke key |
| PATCH | /merchants/me/webhook | JWT | Set webhook URL |
| POST | /v1/checkout/sessions | API Key | Create session |
| GET | /v1/checkout/sessions/:id | None | Get session status |
| POST | /v1/checkout/sessions/:id/cancel | API Key | Cancel session |
| GET | /health | None | Health check |
| GET | /metrics | None | Prometheus metrics |
