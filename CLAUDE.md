# Stellar Checkout Documentation

## Project
Documentation for Stellar Checkout — a Stripe-like merchant payment gateway for Stellar.

## Structure
architecture/
  overview.md              - system architecture
  backend.md               - backend service design
  frontend.md              - frontend design
  contract.md              - escrow contract design
api/
  openapi.yaml             - REST API specification
contract/
  spec.md                  - escrow contract functions
  events.md                - contract events
guides/
  integration-guide.md     - how to integrate Stellar Checkout
  merchant-setup.md        - how to register and configure
security/
  threat-model.md          - security considerations

## Rules
- Keep spec.md in sync with actual contract at all times
- Keep openapi.yaml in sync with actual API at all times
- Every contract function must have an example in spec.md
- Docs are written for a technical audience
