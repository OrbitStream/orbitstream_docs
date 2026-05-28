# OrbitStream Documentation

## Project
Documentation for OrbitStream — a Stripe-like merchant payment gateway for Stellar.
Uses Stellar-native features: SEP protocols, built-in DEX, muxed accounts, claimable balances.

## Structure
architecture/
  overview.md              - system architecture
  backend.md               - backend service design
  frontend.md              - frontend design + embeddable widget
  contract.md              - escrow architecture (Claimable Balances + Soroban)
  stellar-features.md      - SEP protocols, DEX, muxed accounts, claimable balances
api/
  openapi.yaml             - REST API specification
contract/
  spec.md                  - escrow contract functions (Soroban + Claimable Balances)
  events.md                - contract events
guides/
  integration-guide.md     - SDK, widget, and webhook integration
  merchant-setup.md        - registration, API keys, fiat settlement
  roadmap.md               - MVP phases and timeline
  competitive.md           - competitive landscape analysis
security/
  threat-model.md          - security considerations (incl. SEP, DEX, escrow)

## Rules
- Keep spec.md in sync with actual contract at all times
- Keep openapi.yaml in sync with actual API at all times
- Every contract function must have an example in spec.md
- Docs are written for a technical audience
- Use "OrbitStream" as the product name (not "Stellar Checkout")
