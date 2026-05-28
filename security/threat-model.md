# Threat Model

## Payment Detection

### Threat: Fake Payment Notification
**Attack:** Attacker sends a payment notification with a spoofed memo to claim a session.
**Mitigation:** Backend verifies payments by querying Horizon directly. Never trust client-reported payment data.

### Threat: Double-Spend
**Attack:** Attacker submits a payment that gets reversed.
**Mitigation:** Wait for Stellar's 5-second finality before confirming. Verify transaction success on Horizon.

### Threat: Memo Collision
**Attack:** Two sessions get the same memo, causing misattribution.
**Mitigation:** Memos are 16-character hex strings (64 bits). Collision probability is negligible.

## Webhooks

### Threat: Webhook Spoofing
**Attack:** Attacker sends fake webhook events to merchant endpoint.
**Mitigation:** All webhooks signed with HMAC-SHA256. Merchant must verify signature before processing.

### Threat: Webhook Replay
**Attack:** Attacker replays a valid webhook.
**Mitigation:** Timestamp included in signed payload. Merchant should reject events older than 5 minutes.

## API Keys

### Threat: Key Theft
**Attack:** Attacker obtains merchant's API key.
**Mitigation:** Keys shown once at creation. Merchant can revoke compromised keys. API key only allows session creation, not fund access.

### Threat: Brute Force
**Attack:** Attacker tries to guess API keys.
**Mitigation:** Keys are 48+ random hex characters. Rate limiting on all endpoints.

## Merchant Dashboard

### Threat: Wallet Impersonation
**Attack:** Attacker connects a different wallet to access another merchant's dashboard.
**Mitigation:** JWT is bound to wallet address. Dashboard only shows data for the authenticated wallet.

## Escrow Contract

### Threat: Premature Refund
**Attack:** Buyer tries to refund before timeout.
**Mitigation:** Contract checks `env.ledger().timestamp() < escrow.timeout_at` and rejects.

### Threat: Unauthorized Release
**Attack:** Non-seller tries to release funds.
**Mitigation:** `escrow.seller.require_auth()` enforced by Soroban.
