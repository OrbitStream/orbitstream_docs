# Stellar-Specific Features

OrbitStream is Stellar-native. It uses protocols and primitives built into the Stellar network rather than reinventing them.

## SEP Protocol Integration

Stellar Ecosystem Proposals (SEPs) are standardized protocols that OrbitStream uses for authentication, compliance, and fiat settlement.

| SEP | Name | How OrbitStream Uses It |
|-----|------|------------------------|
| [SEP-10](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0010.md) | Stellar Authentication | Merchants and customers prove Stellar account ownership by signing a challenge transaction. Used for wallet-based login and API authentication. |
| [SEP-12](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0012.md) | KYC API | Collects KYC data from merchants required by anchors for fiat settlement. Orchestrated via the anchor's SEP-12 endpoint. |
| [SEP-24](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0024.md) | Hosted Deposit/Withdrawal | Provides fiat on/off ramp via an anchor-hosted iframe. Merchants withdraw USDC to local currency (USD, EUR, ARS, etc.) through their linked anchor. |
| [SEP-31](https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0031.md) | Cross-Border Payments | Enables international merchant settlement. A merchant in one country can receive payments settled in another currency via the anchor network. |

### Fiat Settlement Flow (SEP-24)

```
Merchant                 OrbitStream              Anchor
   |                          |                      |
   |  Request fiat withdrawal |                      |
   |------------------------->|                      |
   |                          |  Initiate SEP-24     |
   |                          |  deposit request     |
   |                          |--------------------->|
   |                          |                      |
   |                          |  Anchor iframe URL   |
   |                          |<---------------------|
   |  Redirect to anchor      |                      |
   |<-------------------------|                      |
   |                          |                      |
   |  Complete KYC (SEP-12)   |                      |
   |--------------------------------------------->  |
   |                          |                      |
   |  Anchor processes fiat   |                      |
   |  withdrawal              |                      |
   |                          |  SEP-24 callback     |
   |                          |<---------------------|
   |  Fiat deposited to bank  |                      |
```

## Muxed Accounts

Stellar muxed accounts (M-accounts) are sub-accounts of a single Stellar address. Each checkout session can use a unique muxed account, making it trivial to match incoming payments to specific orders without on-chain state.

**How it works:**

1. Merchant's Stellar account: `GABC...`
2. Checkout session creates muxed account: `MABC...?memo=1` (unique per session)
3. Customer sends payment to the muxed account
4. OrbitStream identifies the session by the muxed account address

**Advantages over memo-based matching:**

- No memo required — the destination address itself identifies the session
- No risk of memo collision or customer forgetting to include the memo
- Compatible with wallets that don't support custom memos

OrbitStream supports both memo-based and muxed-account payment detection. Memo-based is the default for simplicity; muxed accounts are available for merchants who want a frictionless customer experience.

## Built-in DEX

Stellar has a native decentralized exchange. OrbitStream uses it for:

| Use Case | How It Works |
|----------|-------------|
| **Multi-asset acceptance** | Customer pays in any Stellar asset (XLM, USDC, EURC, etc.). OrbitStream auto-converts to the merchant's preferred currency via path payments. |
| **Fiat price display** | Show checkout amounts in local fiat (USD, EUR, ARS) using DEX order book prices. |
| **Best-rate routing** | Route payments through the best available path on the DEX to minimize slippage. |

```
Customer pays XLM  →  DEX path payment  →  Merchant receives USDC
```

The DEX is on-chain and non-custodial — OrbitStream never holds customer funds during conversion.

## Claimable Balances

Claimable Balances are a Stellar primitive that holds funds with conditional release. OrbitStream uses them for escrow-like flows without requiring a smart contract.

**How it works:**

1. Buyer creates a Claimable Balance with the seller as claimant
2. Funds are locked on-chain until the claim conditions are met
3. Seller claims the balance after delivering the goods/service
4. If timeout passes unclaimed, buyer can reclaim the funds

**When to use Claimable Balances vs. Soroban Escrow:**

| Feature | Claimable Balances | Soroban Escrow |
|---------|-------------------|----------------|
| Complexity | Low — native Stellar operation | Higher — requires contract deployment |
| Flexibility | Basic: claimant + timeout | Full: custom logic, multi-party, dispute resolution |
| Gas cost | Minimal | Contract execution fees |
| Best for | Simple marketplace transactions | Complex escrow with arbitration |

OrbitStream defaults to Claimable Balances for simple escrow flows and offers Soroban contracts for advanced use cases.
