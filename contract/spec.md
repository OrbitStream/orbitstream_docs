# Escrow Contract Specification

## Escrow Functions (Soroban)

These functions manage escrowed payments for marketplace/freelance transactions using Soroban smart contracts. Use these when you need custom logic, multi-party escrow, or dispute resolution.

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

---

## Claimable Balance Functions (Stellar Classic)

These functions use Stellar's native Claimable Balances for simple escrow flows without deploying a smart contract. Best for straightforward buyer-seller transactions where a timeout-based refund is sufficient.

### create_claimable_escrow

Creates a Claimable Balance locked to the seller with a timeout for buyer refund.

**Parameters:**
| Name | Type | Description |
|------|------|-------------|
| buyer | Address | Buyer's Stellar address (source of funds, must sign) |
| seller | Address | Seller's Stellar address (claimant) |
| asset | Asset | Stellar asset to escrow (e.g. USDC, XLM) |
| amount | i64 | Amount to escrow (in stroops for native, or smallest unit for issued assets) |
| timeout_hours | u32 | Hours until buyer can reclaim |

**Returns:** `String` — Claimable Balance ID

**Behavior:**
1. Creates a Claimable Balance with two claimants:
   - **Seller:** can claim immediately (unconditional predicate)
   - **Buyer:** can claim after timeout (`after` predicate using absolute Unix timestamp)
2. Funds are locked on-chain until claimed or timeout expires

**Errors:**
- `InvalidAmount` — amount is 0 or negative
- `InvalidTimeout` — timeout is 0
- `InsufficientBalance` — buyer lacks funds

**Example:**
```typescript
const balanceId = await orbitstream.createClaimableEscrow({
  buyer: 'GBUYER...',
  seller: 'GSELLER...',
  asset: 'USDC',
  amount: 10000000, // 10 USDC (7 decimals)
  timeoutHours: 72,
});
// balanceId = "00000000abcdef1234567890..."
```

---

### claim_escrow

Claims a Claimable Balance. Used by the seller after delivering goods/services, or by the buyer after timeout.

**Parameters:**
| Name | Type | Description |
|------|------|-------------|
| claimant | Address | Address claiming the balance (must sign) |
| balance_id | String | Claimable Balance ID from `create_claimable_escrow` |

**Returns:** `()` — funds transferred to claimant

**Behavior:**
- If called by **seller** (before timeout): seller receives funds
- If called by **buyer** (after timeout): buyer receives refund
- Claimable Balance is consumed after first successful claim

**Errors:**
- `BalanceNotFound` — balance ID doesn't exist
- `NotClaimant` — caller is not a claimant on this balance
- `TimeoutNotReached` — buyer trying to claim before timeout

**Example:**
```typescript
// Seller claims after delivering goods
await orbitstream.claimEscrow({
  claimant: 'GSELLER...',
  balanceId: '00000000abcdef1234567890...',
});

// Buyer claims refund after timeout
await orbitstream.claimEscrow({
  claimant: 'GBUYER...',
  balanceId: '00000000abcdef1234567890...',
});
```

---

### get_claimable_escrow

Returns the status and details of a Claimable Balance escrow.

**Parameters:**
| Name | Type | Description |
|------|------|-------------|
| balance_id | String | Claimable Balance ID |

**Returns:** Claimable balance details (amount, asset, claimants, predicates)

**Errors:**
- `BalanceNotFound` — balance doesn't exist or was already claimed

**Example:**
```typescript
const escrow = await orbitstream.getClaimableEscrow({
  balanceId: '00000000abcdef1234567890...',
});
// escrow.status: 'active' | 'claimed' | 'expired'
// escrow.amount: 10000000
// escrow.asset: 'USDC'
```

### When to Use Claimable Balances vs. Soroban Escrow

| Scenario | Use |
|----------|-----|
| Simple marketplace transaction | Claimable Balances |
| Need custom release conditions | Soroban Escrow |
| Multi-party escrow (arbiter) | Soroban Escrow |
| Subscription/recurring payments | Soroban Escrow |
| Minimal gas cost | Claimable Balances |
| No contract deployment needed | Claimable Balances |
