# Frontend Architecture

## Stack
- Next.js 16 (App Router)
- TypeScript
- Tailwind CSS + shadcn/ui
- SWR for data fetching
- @stellar/stellar-sdk + Freighter

## Pages

### Landing Page (`/`)
Merchant-facing marketing page with features, how-it-works, and CTA.

### Checkout Page (`/checkout/[sessionId]`)
Customer-facing payment page. Fetches session from API, displays:
- Amount and asset
- QR code (web+stellar:pay? URI)
- Freighter wallet connect + pay button
- Payment status (polls every 3s while pending)
- Confirmation screen on success

### Merchant Dashboard (`/merchant`)
- Wallet login
- Stats overview (sessions, payments, revenue)
- Recent payments list

### Merchant Settings (`/merchant/settings`)
- API key management
- Webhook URL configuration

## Components

### checkout/AssetSelector
Radio group for USDC/XLM selection.

### checkout/QRCode
Renders QR code from Stellar payment URI. Shows copyable receiving address.

### checkout/WalletConnect
Freighter wallet connect button. Shows connected address and "Pay" button.

### checkout/PaymentStatus
Displays session state: waiting (spinner), confirmed (green check), expired (red X), cancelled.

### checkout/CheckoutForm
Composes all checkout components. Main checkout page content.

## Hooks

### useCheckout(sessionId)
SWR hook that fetches session status. Polls every 3s while pending. Returns session, isLoading, isPaid, isExpired.

### useWallet()
Freighter wallet hook. Returns address, connect, disconnect, signTransaction, isConnected.

## Libraries

### lib/api
Backend API client. fetchSession, registerMerchant, merchantLogin.

### lib/stellar
Stellar SDK setup. Horizon server instance, network passphrase.

### lib/stellar-uri
buildStellarPayURI() — constructs web+stellar:pay? URI for QR codes.
