# Escrow Contract Architecture

## Purpose
Manages escrowed payments for marketplace and freelance transactions. Buyers deposit funds that are locked until released by seller or refunded after timeout.

## Contract Functions

### create_escrow(buyer, seller, token, amount, timeout_seconds) -> escrow_id
- Requires buyer auth
- Validates amount > 0 and timeout > 0
- Creates escrow record with Active status
- Emits EscrowCreated event
- Returns unique escrow_id

### release(escrow_id)
- Requires seller auth
- Escrow must be Active
- Sets status to Released
- Emits EscrowReleased event

### refund(escrow_id)
- Requires buyer auth
- Escrow must be Active
- Current time must be past timeout_at
- Sets status to Refunded
- Emits EscrowRefunded event

### get_escrow(escrow_id) -> Escrow
- Read-only
- Returns escrow details or EscrowNotFound

## Data Model

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

## Error Codes
1. EscrowNotFound
2. Unauthorized
3. EscrowAlreadySettled
4. TimeoutNotReached
5. InvalidAmount
6. InvalidTimeout

## Design Decisions
- No smart contract token transfer in Phase 1 — escrow is a record-keeping contract
- Timeout is ledger-timestamp based (seconds since epoch)
- Only buyer can create and refund; only seller can release
- Single escrow per transaction (no batching)
