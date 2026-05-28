# Contract Events

## Soroban Escrow Events

### EscrowCreated

Emitted when a new escrow is created.

**Fields:**
| Field | Type | Description |
|-------|------|-------------|
| escrow_id | u64 | Unique escrow identifier |
| buyer | Address | Buyer's Stellar address |
| seller | Address | Seller's Stellar address |
| token | Address | Token contract address |
| amount | u128 | Escrowed amount |
| timeout_at | u64 | Timestamp when refund becomes available |

---

## EscrowReleased

Emitted when seller releases escrowed funds.

**Fields:**
| Field | Type | Description |
|-------|------|-------------|
| escrow_id | u64 | Escrow identifier |
| seller | Address | Seller's Stellar address |
| amount | u128 | Released amount |

---

### EscrowRefunded

Emitted when buyer refunds after timeout.

**Fields:**
| Field | Type | Description |
|-------|------|-------------|
| escrow_id | u64 | Escrow identifier |
| buyer | Address | Buyer's Stellar address |
| amount | u128 | Refunded amount |

---

## Claimable Balance Events (Stellar Classic)

These events are emitted by OrbitStream when managing Claimable Balance escrows. They are indexed from the Stellar ledger, not emitted by a smart contract.

### ClaimableEscrowCreated

Emitted when a Claimable Balance is created for escrow.

**Fields:**
| Field | Type | Description |
|-------|------|-------------|
| balance_id | String | Claimable Balance ID |
| buyer | Address | Buyer's Stellar address (source) |
| seller | Address | Seller's Stellar address (claimant) |
| asset | String | Stellar asset code (e.g. "USDC", "native") |
| amount | i64 | Escrowed amount |
| timeout_at | u64 | Unix timestamp when buyer can reclaim |

---

### ClaimableEscrowClaimed

Emitted when a Claimable Balance is claimed by seller or buyer.

**Fields:**
| Field | Type | Description |
|-------|------|-------------|
| balance_id | String | Claimable Balance ID |
| claimant | Address | Address that claimed (seller or buyer) |
| asset | String | Stellar asset code |
| amount | i64 | Claimed amount |
| claim_type | String | "release" (seller claimed) or "refund" (buyer claimed after timeout) |
