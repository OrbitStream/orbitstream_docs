# Escrow Contract Specification

## Functions

### create_escrow

Creates a new escrow, locking funds from the buyer.

**Parameters:**
| Name | Type | Description |
|------|------|-------------|
| buyer | Address | Buyer's Stellar address (must sign) |
| seller | Address | Seller's Stellar address |
| token | Address | Token contract address |
| amount | u128 | Amount to escrow |
| timeout_seconds | u64 | Seconds until buyer can refund |

**Returns:** `u64` — escrow ID

**Errors:**
- `InvalidAmount` — amount is 0
- `InvalidTimeout` — timeout is 0

**Example:**
```rust
let escrow_id = client.create_escrow(&buyer, &seller, &token, &1000, &3600);
// escrow_id = 1
```

---

### release

Releases escrowed funds to the seller.

**Parameters:**
| Name | Type | Description |
|------|------|-------------|
| escrow_id | u64 | ID of the escrow to release |

**Returns:** `()`

**Errors:**
- `EscrowNotFound` — escrow doesn't exist
- `EscrowAlreadySettled` — already released or refunded
- `Unauthorized` — caller is not the seller

**Example:**
```rust
client.release(&escrow_id);
```

---

### refund

Refunds escrowed funds to the buyer after timeout.

**Parameters:**
| Name | Type | Description |
|------|------|-------------|
| escrow_id | u64 | ID of the escrow to refund |

**Returns:** `()`

**Errors:**
- `EscrowNotFound` — escrow doesn't exist
- `EscrowAlreadySettled` — already released or refunded
- `TimeoutNotReached` — timeout hasn't passed yet
- `Unauthorized` — caller is not the buyer

**Example:**
```rust
// Advance ledger time past timeout
client.refund(&escrow_id);
```

---

### get_escrow

Returns escrow details.

**Parameters:**
| Name | Type | Description |
|------|------|-------------|
| escrow_id | u64 | ID of the escrow |

**Returns:** `Escrow` struct

**Errors:**
- `EscrowNotFound` — escrow doesn't exist

**Example:**
```rust
let escrow = client.get_escrow(&1);
assert_eq!(escrow.status, EscrowStatus::Active);
```
