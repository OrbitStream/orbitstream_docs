# Contract Events

## EscrowCreated

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

## EscrowRefunded

Emitted when buyer refunds after timeout.

**Fields:**
| Field | Type | Description |
|-------|------|-------------|
| escrow_id | u64 | Escrow identifier |
| buyer | Address | Buyer's Stellar address |
| amount | u128 | Refunded amount |
