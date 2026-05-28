# Integration Guide

## Quick Start

### 1. Install the SDK

```bash
npm install @orbitstream/sdk
```

### 2. Create a Checkout Session

```typescript
import { OrbitStream } from '@orbitstream/sdk';

const orbitstream = new OrbitStream({ apiKey: 'sk_test_...' });

const session = await orbitstream.createSession({
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
app.post('/webhooks/orbitstream', (req, res) => {
  const signature = req.headers['x-orbitstream-signature'];
  const timestamp = req.headers['x-orbitstream-timestamp'];

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

Redirect customers to an OrbitStream hosted page. No frontend code needed.

```typescript
const session = await orbitstream.createSession({
  amount: 25.00,
  asset: 'USDC',
  successUrl: 'https://example.com/success',
  cancelUrl: 'https://example.com/cancel',
});

// Redirect to OrbitStream-hosted checkout
window.location.href = session.url;
```

### Embeddable Widget

Embed checkout directly on your page with a `<script>` tag. Uses Web Components for framework-agnostic integration — works with React, Vue, Svelte, or plain HTML.

```html
<!-- Add the script -->
<script src="https://js.orbitstream.dev/v1/widget.js"></script>

<!-- Place the widget -->
<orbitstream-checkout
  session-id="cs_abc123"
  theme="light"
></orbitstream-checkout>
```

**React component:**

```tsx
import { OrbitStreamCheckout } from '@orbitstream/sdk/react';

function PayPage({ sessionId }: { sessionId: string }) {
  return (
    <OrbitStreamCheckout
      sessionId={sessionId}
      theme="light"
      onPaymentConfirmed={(txHash) => {
        console.log('Paid!', txHash);
      }}
    />
  );
}
```

### SDK Integration

For full control, use the SDK to create sessions and handle the flow yourself.

```typescript
import { OrbitStream } from '@orbitstream/sdk';

const orbitstream = new OrbitStream({ apiKey: 'sk_test_...' });

// Create session
const session = await orbitstream.createSession({
  amount: 25.00,
  asset: 'USDC',
  successUrl: 'https://example.com/success',
  cancelUrl: 'https://example.com/cancel',
});

// Poll for status
const status = await orbitstream.getSession(session.id);
// status: 'pending' | 'confirmed' | 'expired' | 'cancelled'
```

## Multi-Asset Support

Customers can pay in any Stellar asset (XLM, USDC, EURC, etc.). OrbitStream auto-converts to the merchant's preferred currency via Stellar's built-in DEX.

```typescript
const session = await orbitstream.createSession({
  amount: 25.00,
  asset: 'USDC',           // Merchant receives USDC
  acceptAssets: ['XLM', 'USDC', 'EURC'],  // Customer can pay in any of these
  successUrl: 'https://example.com/success',
  cancelUrl: 'https://example.com/cancel',
});
```

The checkout page shows the equivalent amount in each accepted asset using real-time DEX prices.

## Webhook Events

| Event | Description |
|-------|-------------|
| payment.confirmed | Payment detected and confirmed on Stellar |
| payment.expired | Checkout session expired without payment |

### Webhook Payload

```json
{
  "event": "payment.confirmed",
  "data": {
    "sessionId": "uuid",
    "txHash": "abc123...",
    "amount": "25.00",
    "asset": "USDC",
    "sender": "GABC...",
    "memo": "a1b2c3d4e5f67890"
  },
  "timestamp": "2026-01-15T10:30:00Z"
}
```

### Signature Verification

Webhooks are signed with HMAC-SHA256 using your webhook secret.

```
X-OrbitStream-Signature: <hex digest>
X-OrbitStream-Timestamp: <ISO timestamp>
```

Verify the signature before processing any webhook event. See [Merchant Setup](merchant-setup.md#4-configure-webhook) for how to get your webhook secret.
