# Escrow Contract Architecture

## Purpose

OrbitStream provides two escrow mechanisms for marketplace and freelance transactions:

1. **Claimable Balances** (Stellar Classic) — simple, low-cost escrow for straightforward buyer-seller transactions
2. **Soroban Escrow Contract** — programmable escrow for complex flows with custom logic, multi-party arbitration, or dispute resolution

## Claimable Balances (Default for Simple Escrow)

Stellar's native Claimable Balances provide escrow-like functionality without deploying a smart contract.

### Flow

```
Buyer                    Stellar Network              Seller
  |                            |                          |
  |  Create Claimable Balance  |                          |
  |  (seller as claimant,      |                          |
  |   buyer after timeout)     |                          |
  |--------------------------->|                          |
  |                            |                          |
  |  Funds locked on-chain     |                          |
  |                            |                          |
  |                            |  Seller delivers goods   |
  |                            |                          |
  |                            |  Seller claims balance   |
  |                            |<-------------------------|
  |                            |                          |
  |  If timeout passes:        |                          |
  |  Buyer reclaims funds      |                          |
  |<---------------------------|                          |
```

### Functions

- `create_claimable_escrow(buyer, seller, asset, amount, timeout_hours)` — creates Claimable Balance with seller as immediate claimant, buyer as post-timeout claimant
- `claim_escrow(claimant, balance_id)` — seller claims immediately, buyer claims after timeout
- `get_claimable_escrow(balance_id)` — returns balance status and details

### When to Use

- Simple marketplace transactions
- Minimal gas cost needed
- No custom release conditions
- No contract deployment required

## Soroban Escrow Contract (Advanced)

For complex escrow flows requiring custom logic, multi-party arbitration, or programmable conditions.

### Contract Functions

#### create_escrow(buyer, seller, token, amount, timeout_seconds) -> escrow_id
- Requires buyer auth
- Validates amount > 0 and timeout > 0
- Creates escrow record with Active status
- Emits EscrowCreated event
- Returns unique escrow_id

#### release(escrow_id)
- Requires seller auth
- Escrow must be Active
- Sets status to Released
- Emits EscrowReleased event

#### refund(escrow_id)
- Requires buyer auth
- Escrow must be Active
- Current time must be past timeout_at
- Sets status to Refunded
- Emits EscrowRefunded event

#### get_escrow(escrow_id) -> Escrow
- Read-only
- Returns escrow details or EscrowNotFound

### Data Model

```rust
pub struct Escrow {
    pub id: u64,
    pub buyer: Address,
    pub seller: Address,
    pub token: Address,
    pub amount: u128,
    pub status: EscrowStatus,
    pub created_at: u64,
    pub timeout_at: u64,
}

pub enum EscrowStatus {
    Active = 0,
    Released = 1,
    Refunded = 2,
}
```

### Error Codes
1. EscrowNotFound
2. Unauthorized
3. EscrowAlreadySettled
4. TimeoutNotReached
5. InvalidAmount
6. InvalidTimeout

## Design Decisions
- Claimable Balances are the default escrow mechanism for simplicity and cost efficiency
- Soroban contract is opt-in for advanced use cases
- No smart contract token transfer in Phase 1 — escrow is a record-keeping contract
- Timeout is ledger-timestamp based (seconds since epoch)
- Only buyer can create and refund; only seller can release
- Single escrow per transaction (no batching)
