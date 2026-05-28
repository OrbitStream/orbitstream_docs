# Frontend Architecture

## Stack
- Next.js 16 (App Router)
- TypeScript
- Tailwind CSS + shadcn/ui
- SWR for data fetching
- @stellar/stellar-sdk + Freighter
- Web Components (for embeddable widget)

## Pages

### Landing Page (`/`)
Merchant-facing marketing page with features, how-it-works, and CTA.

### Checkout Page (`/checkout/[sessionId]`)
Customer-facing payment page. Fetches session from API, displays:
- Amount and asset (merchant's preferred currency)
- **Multi-asset selector** — shows equivalent amounts in each accepted asset using real-time DEX prices
- QR code (web+stellar:pay? URI)
- Freighter wallet connect + pay button
- Payment status (polls every 3s while pending)
- Confirmation screen on success

### Merchant Dashboard (`/merchant`)
- Wallet login (SEP-10 authentication)
- Stats overview (sessions, payments, revenue)
- Recent payments list
- Fiat settlement overview (connected anchor, withdrawal history)

### Merchant Settings (`/merchant/settings`)
- API key management
- Webhook URL configuration
- Anchor connection for fiat settlement

## Components

### checkout/AssetSelector
Radio group for asset selection. When `acceptAssets` is configured, shows all accepted assets with real-time DEX prices. Displays equivalent amounts so customers can choose which asset to pay with.

### checkout/QRCode
Renders QR code from Stellar payment URI. Shows copyable receiving address. Adapts URI based on selected asset.

### checkout/WalletConnect
Freighter wallet connect button. Shows connected address and "Pay" button. Signs the appropriate payment transaction based on selected asset.

### checkout/PaymentStatus
Displays session state: waiting (spinner), confirmed (green check), expired (red X), cancelled.

### checkout/CheckoutForm
Composes all checkout components. Main checkout page content.

## Embeddable Widget (Web Components)

The embeddable widget is a framework-agnostic Web Component that can be dropped into any page with a `<script>` tag.

### Architecture

```
<script src="https://js.orbitstream.dev/v1/widget.js"></script>
<orbitstream-checkout session-id="cs_abc123" theme="light"></orbitstream-checkout>
```

The widget:
- Is self-contained (loads its own CSS, no external dependencies)
- Works with React, Vue, Svelte, Angular, or plain HTML
- Communicates with the OrbitStream API directly
- Supports custom themes via CSS custom properties
- Emits events: `payment-confirmed`, `payment-expired`, `error`

### Props

| Prop | Type | Description |
|------|------|-------------|
| session-id | string | Checkout session ID |
| theme | 'light' \| 'dark' | Color theme (default: 'light') |
| on-payment-confirmed | function | Callback with txHash |
| on-error | function | Callback with error details |

### React Wrapper

A React component wrapper (`@orbitstream/sdk/react`) provides type-safe props and event handling.

## Hooks

### useCheckout(sessionId)
SWR hook that fetches session status. Polls every 3s while pending. Returns session, isLoading, isPaid, isExpired.

### useWallet()
Freighter wallet hook. Returns address, connect, disconnect, signTransaction, isConnected.

### useDexPrices(assets)
Fetches real-time DEX prices for a list of Stellar assets. Returns conversion rates relative to a base currency. Used by AssetSelector to display equivalent amounts.

## Libraries

### lib/api
Backend API client. fetchSession, registerMerchant, merchantLogin, connectAnchor.

### lib/stellar
Stellar SDK setup. Horizon server instance, network passphrase, path payment helpers for DEX routing.

### lib/stellar-uri
buildStellarPayURI() — constructs web+stellar:pay? URI for QR codes. Supports asset-specific URIs.

### lib/dex
DEX price fetching and conversion. Queries Horizon order books for real-time rates. Caches prices for 30s to reduce API calls.
