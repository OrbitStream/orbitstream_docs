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

## SEP Protocol Integration

### Threat: SEP-10 Challenge Replay

**Attack:** Attacker replays a captured SEP-10 challenge to impersonate a merchant.
**Mitigation:** Challenge transactions include a nonce and are bound to OrbitStream's domain. Expired challenges are rejected.

### Threat: SEP-12 Data Leak

**Attack:** KYC data intercepted during fiat settlement onboarding.
**Mitigation:** KYC data is sent directly to the anchor's SEP-12 endpoint over TLS. OrbitStream never stores KYC documents.

### Threat: SEP-24 Iframe Phishing

**Attack:** Merchant injects a malicious URL as the anchor iframe target.
**Mitigation:** Anchor iframe URLs are fetched server-side from the anchor's TOML file. Merchants cannot inject arbitrary URLs.

### Threat: Malicious Anchor

**Attack:** A fraudulent anchor intercepts fiat settlement funds.
**Mitigation:** OrbitStream validates anchor TOML files and checks SEP compliance before enabling fiat settlement for an anchor.

## Muxed Accounts

### Threat: Muxed Account Spoofing

**Attack:** Attacker creates a muxed account that routes to their own master account.
**Mitigation:** Muxed accounts are derived from the merchant's Stellar account. Only the merchant's master account can authorize withdrawals.

### Threat: Session Hijacking via Muxed ID

**Attack:** Attacker reuses a muxed account ID from a previous session.
**Mitigation:** Muxed IDs are single-use and expire with the checkout session.

## Claimable Balances

### Threat: Premature Claim

**Attack:** Seller claims funds before delivering goods.
**Mitigation:** Claimable balance predicates enforce the timeout at the protocol level. No off-chain check needed.

### Threat: Double-Claim

**Attack:** Claimant tries to claim the same balance twice.
**Mitigation:** Stellar protocol guarantees a claimable balance can only be claimed once.

### Threat: Stale Balance

**Attack:** Funds locked indefinitely in an unclaimed balance.
**Mitigation:** Unclaimed balances are automatically returned to the creator after the predicate timeout.

## Built-in DEX

### Threat: Price Manipulation

**Attack:** Attacker manipulates DEX order book to change the conversion rate mid-checkout.
**Mitigation:** DEX prices are fetched at session creation. The quoted amount is locked for the session duration.

### Threat: Slippage

**Attack:** DEX rate changes between quote and execution, causing the merchant to receive less than expected.
**Mitigation:** Path payments use strict send/receive amounts. If the rate changes beyond a configurable threshold, the payment fails safely.

## Escrow Contract

### Threat: Premature Refund

**Attack:** Buyer tries to refund before timeout.
**Mitigation:** Contract checks `env.ledger().timestamp() < escrow.timeout_at` and rejects.

### Threat: Unauthorized Release

**Attack:** Non-seller tries to release funds.
**Mitigation:** `escrow.seller.require_auth()` enforced by Soroban.
