# Integration Guide

## Quick Start

### 1. Install the SDK

```bash
npm install @stellar-checkout/sdk
```

### 2. Create a Checkout Session

```typescript
import { StellarCheckout } from '@stellar-checkout/sdk';

const checkout = new StellarCheckout({ apiKey: 'sk_test_...' });

const session = await checkout.createSession({
  amount: 25.00,
  asset: 'USDC',
  successUrl: 'https://example.com/success',
  cancelUrl: 'https://example.com/cancel',
});

// Redirect customer to checkout
window.location.href = session.url;
```

### 3. Listen for Webhooks

```javascript
app.post('/webhooks/stellar', (req, res) => {
  const signature = req.headers['x-stellar-checkout-signature'];
  const timestamp = req.headers['x-stellar-checkout-timestamp'];

  // Verify HMAC signature
  const expected = crypto
    .createHmac('sha256', webhookSecret)
    .update(JSON.stringify(req.body))
    .digest('hex');

  if (signature !== expected) {
    return res.status(401).send('Invalid signature');
  }

  const { event, data } = req.body;

  if (event === 'payment.confirmed') {
    console.log('Payment received:', data.txHash);
    // Fulfill order
  }

  res.status(200).send('OK');
});
```

## Integration Modes

### Hosted Checkout (Recommended)
Redirect customers to a Stellar Checkout hosted page. No frontend code needed.

### Embedded Widget (Coming in Phase 2)
Embed checkout directly on your page with a `<script>` tag.

## Webhook Events

| Event | Description |
|-------|-------------|
| payment.confirmed | Payment detected and confirmed on Stellar |

### Webhook Payload

```json
{
  "event": "payment.confirmed",
  "data": {
    "sessionId": "uuid",
    "txHash": "abc123...",
    "amount": "25.00",
    "asset": "USDC",
    "sender": "GABC..."
  },
  "timestamp": "2026-01-15T10:30:00Z"
}
```

### Signature Verification

Webhooks are signed with HMAC-SHA256 using your webhook secret.

```
X-Stellar-Checkout-Signature: <hex digest>
X-Stellar-Checkout-Timestamp: <ISO timestamp>
```
