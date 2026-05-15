# Smart Contract Reference

**Contract:** `orbitstream-stream`  
**Language:** Rust (Soroban SDK 25)  
**Network:** Stellar Testnet / Mainnet

---

## Data Types

### `StreamStatus`

```rust
enum StreamStatus {
    Active,      // Clock running, tokens accruing
    Paused,      // Clock frozen, no accrual
    Cancelled,   // Terminated early, both parties settled
    Completed,   // Reached natural end time
}
```

### `Stream`

```rust
struct Stream {
    id:                   u64,
    sender:               Address,
    recipient:            Address,
    token:                Address,      // Stellar asset contract
    rate_per_second:      i128,         // Token units per active second
    start_time:           u64,          // Unix timestamp
    end_time:             u64,          // 0 = open-ended
    total_deposited:      i128,
    total_claimed:        i128,
    status:               StreamStatus,
    paused_at:            u64,          // 0 if not paused
    total_paused_seconds: u64,
}
```

---

## Functions

### `initialize(env, admin)`

One-time setup. Must be called before any other function.

```bash
stellar contract invoke --id $CONTRACT_ID -- initialize --admin $ADMIN
```

---

### `create_stream(env, sender, recipient, token, rate_per_second, duration_seconds, deposit) → u64`

Opens a new stream. Locks `deposit` tokens immediately. Returns the stream ID.

| Parameter | Type | Notes |
|-----------|------|-------|
| `sender` | Address | Must sign. Pays the deposit. |
| `recipient` | Address | Cannot equal sender |
| `token` | Address | Stellar asset contract ID |
| `rate_per_second` | i128 | Must be > 0 |
| `duration_seconds` | u64 | 0 = open-ended |
| `deposit` | i128 | Must be ≥ rate_per_second |

```bash
stellar contract invoke --id $CONTRACT_ID -- create_stream \
  --sender $SENDER \
  --recipient $RECIPIENT \
  --token $TOKEN \
  --rate_per_second 347222 \
  --duration_seconds 2592000 \
  --deposit 1000000000
```

---

### `top_up(env, sender, stream_id, amount) → i128`

Adds more tokens to an active or paused stream. Extends runway without restarting the stream. Returns new `total_deposited`.

---

### `claim(env, recipient, stream_id) → i128`

Transfers all accrued tokens to the recipient. Returns the payout amount.

**Claimable formula:**
```
active_seconds  = (now - start_time) - total_paused_seconds
earned          = active_seconds × rate_per_second
claimable       = min(earned - total_claimed, total_deposited - total_claimed)
```

---

### `pause_stream(env, sender, stream_id)`

Freezes the clock. Requires `status == Active`. No tokens accrue while paused.

---

### `resume_stream(env, sender, stream_id)`

Resumes the stream. Adds `(now - paused_at)` to `total_paused_seconds`. Requires `status == Paused`.

---

### `cancel_stream(env, sender, stream_id)`

Cancels the stream. Settlement:
1. Recipient receives all earned-but-unclaimed tokens.
2. Sender receives all unearned tokens.

Works on both `Active` and `Paused` streams.

---

### `admin_cancel(env, stream_id)`

Emergency cancel by the admin. Same settlement as `cancel_stream`. Admin must have been set via `initialize`.

---

### `get_stream(env, stream_id) → Option<Stream>`

Returns the full stream struct, or `None` if not found.

---

### `claimable_amount(env, stream_id) → i128`

Read-only. Returns tokens the recipient can claim right now without modifying state.

---

### `active_seconds(env, stream_id) → u64`

Returns the number of seconds the stream has actively run (excluding paused time).

---

### `stream_count(env) → u64`

Returns the total number of streams ever created.

---

## Error Codes

| Code | Name | Description |
|------|------|-------------|
| 1 | `AlreadyInitialized` | `initialize` already called |
| 2 | `NotInitialized` | Contract not set up |
| 3 | `Unauthorized` | Caller doesn't own this stream |
| 4 | `NotFound` | Stream ID doesn't exist |
| 5 | `InvalidRate` | `rate_per_second` must be > 0 |
| 6 | `InvalidDeposit` | `deposit` must be > 0 |
| 7 | `SelfStream` | Sender and recipient are the same address |
| 8 | `StreamNotActive` | Stream must be Active |
| 9 | `StreamNotPaused` | Stream must be Paused to resume |
| 10 | `NothingToClaim` | No tokens have accrued since last claim |
| 11 | `AlreadyTerminated` | Stream is Cancelled or Completed |
| 12 | `InsufficientFunds` | Deposit doesn't cover one second of streaming |

---

## Events

| Event | Data | Trigger |
|-------|------|---------|
| `created` | `(id, sender, recipient, rate, deposit)` | `create_stream` |
| `topup` | `(stream_id, amount)` | `top_up` |
| `claimed` | `(stream_id, recipient, payout)` | `claim` |
| `paused` | `(stream_id,)` | `pause_stream` |
| `resumed` | `(stream_id,)` | `resume_stream` |
| `cancel` | `(stream_id, recipient_payout, sender_refund)` | `cancel_stream` |
| `admcancel` | `(stream_id,)` | `admin_cancel` |

---

## Storage TTLs

| Key Type | TTL | ~Duration |
|----------|-----|-----------|
| Instance (admin, count) | 518,400 ledgers | 30 days |
| Stream records | 6,307,200 ledgers | 1 year |

TTLs are extended on every write operation.
