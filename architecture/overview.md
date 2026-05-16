# OrbitStream — System Architecture

## Overview

OrbitStream is a Stellar-based token streaming payroll platform that enables employers to create time-continuous token streams to pay employees. The system combines on-ledger primitives (Stellar accounts / issued assets / streaming logic) with off-chain backend services that provide APIs, monitoring, reconciliation, and user interfaces.

This document describes the primary components, their responsibilities, data flows, and operational considerations.

## Goals

- Continuous, auditable payroll streams denominated in a Stellar-issued asset.
- On-chain settlement and state anchored in the Stellar ledger.
- Robust off-chain services for UX, indexing, access control, and resiliency.
- Secure key management and clear separation of issuer/distributor roles.

## High-level Components

- On-chain (Stellar):
  - Issuer account and distribution accounts for the payroll asset.
  - Streaming contract logic represented by transactions or Soroban smart contracts (where available).
  - Horizon node and Stellar network as the source of truth for account balances and events.

- Backend services:
  - API service: REST endpoints for employers and employees, authentication, and webhooks.
  - Worker/processor: submits transactions, funds streams, and performs on-chain operations.
  - Indexer/Listener: subscribes to Horizon (or runs a full archive) to index account and payment events, building a queryable representation of streams and claims.
  - Reconciliation service: matches on-chain state to off-chain records and alerts on mismatches.
  - Key management / signer: handles signing of transactions for administrative flows (kept in HSM or secure vault).
  - Storage: PostgreSQL for relational state, Redis (or message queue) for task queueing and pub/sub for real-time events.

- Frontend:
  - Employer dashboard: create/manage streams, fund streams, view payroll schedule and run reports.
  - Employee portal: view incoming streams, claimable balance, and claim history.
  - Wallet integrations: allow employees/employers to connect Stellar wallets (custodial or external) for signing.
  - Realtime layer: WebSocket connections (or server-sent events) to push stream balance updates.

- Integrations:
  - Anchor services for fiat on/off ramps where required.
  - External monitoring and alerting (Prometheus/Grafana, Sentry).

## Data & Control Flows

1. Employer creates a payroll `Stream` via the API (parameters: employer account, employee account, asset, start/end timestamps or rate, total amount).
2. Backend validates request, persists canonical stream record in Postgres, and prepares on-chain funding transaction.
3. Worker constructs and submits Stellar transaction(s) to lock funds and establish the streaming contract on-chain (or record required metadata if using off-chain streaming logic). Transaction is signed with appropriate keys.
4. Indexer observes the resulting on-chain events via Horizon and updates the off-chain index (stream state, last-paid timestamp, claimable amounts).
5. Employee views realtime claimable balance via WebSocket; when claiming, their client triggers an API call. Backend verifies entitlement and either submits an on-chain payout transaction or constructs an on-chain claim transaction that the employee signs.
6. Reconciliation jobs run periodically to compare on-chain balances and transaction history with off-chain stream records and trigger alerts for discrepancies.

## Streaming Model

- Streams are represented as linear flows defined by `rate` (tokens per second) and `start`/`end` timestamps.
- Claimable balance at time t is computed deterministically from the stream parameters; see `contract/math.md` for the canonical formula and edge cases (pauses, partial cancellations).
- On-chain settlement is the ground truth; off-chain index should be recoverable from Horizon data.

## Security & Key Management

- Use separate keys for issuer, distribution, and operational signers.
- Store private keys in hardware wallets or an HSM/secure vault (HashiCorp Vault, cloud KMS).
- Restrict admin operations with RBAC and MFA.
- Validate and rate-limit incoming API requests; implement strong input validation to prevent malformed transaction construction.

## Operational Considerations

- Run a Horizon node (or use a trusted Horizon provider) to reduce dependency on third parties for event streams.
- Ensure idempotent worker logic for transaction submission; use deduplication keys when retrying.
- Design for eventual consistency: clients should rely on indexed events, not immediate finality assertions.
- Backups and migrations for the Postgres schema; maintain a migration plan for contract/schema changes.

## Scaling & Resilience

- Horizontally scale stateless API instances behind a load balancer.
- Run multiple worker instances consuming from a task queue with leader-election or distributed locking for single-writer operations.
- Cache frequently-read data (e.g., recent stream balances) in Redis to reduce DB and indexer load.

## Observability

- Emit structured logs and application metrics (request latency, transaction submission success/failure, reconciliation drift).
- Alert on critical thresholds: failed transaction rate, reconciliation failures, high signer latency.

## Developer & Deployment Notes

- CI runs unit and integration tests, plus contract end-to-end tests against a Stellar testnet or local emulator.
- Use environment-specific configs for keys and Horizon endpoints.
- Document all contract function signatures and example transactions in `contract/spec.md`.

## Assumptions & Open Questions

- Choice of on-chain streaming primitive: transaction-based pattern vs Soroban smart contracts.
- Custody model for employer funds (custodial platform wallet vs employer-managed accounts).

---

(See `contract/spec.md` for function-level details and `contract/math.md` for claimable balance formulas.)
