# Contract Specification

Canonical list of streaming contract functions, parameters, return values, and example calls. These function signatures are the canonical developer-facing spec used by the backend and frontend when constructing or validating on-chain transactions and off-chain records.

---

## Principal Concepts

- `Stream`: a linear token flow defined by `employer`, `employee`, `asset`, `rate` (tokens/sec), `start`, `end`, and `total`.
- `StreamId`: unique identifier for a `Stream` (UUID or on-chain reference).
- `Withdrawn`: amount already claimed from a stream.

---

## Functions

1. CreateStream

- Purpose: create a new payroll stream and reserve funds (off-chain record + on-chain funding transaction).
- Signature:
  - `CreateStream(employer: PublicKey, employee: PublicKey, asset: Asset, rate: Decimal, start: Timestamp, end: Timestamp, total: Decimal) -> StreamId`
- Side-effects: persists stream in Postgres, enqueues funding job, emits `StreamCreated` event.
- Example request (API):

```json
{
  "employer": "G...EMP",
  "employee": "G...EMP2",
  "asset": { "code": "PAY", "issuer": "G...ISS" },
  "rate": "0.0001157407", // tokens/sec (~10 tokens/day)
  "start": 1700000000,
  "end": 1702592000,
  "total": "25920"
}
```

2. FundStream

- Purpose: ensure on-chain funds are allocated (may be combined with `CreateStream` depending on custody model).
- Signature:
  - `FundStream(streamId: StreamId, fundingAccount: PublicKey, amount: Decimal) -> TransactionHash`
- Example (worker): constructs and submits a Stellar payment or escrow-like transaction; returns transaction hash.

3. Claim

- Purpose: employee claims available (claimable) tokens from a stream.
- Signature:
  - `Claim(streamId: StreamId, claimant: PublicKey, destination: PublicKey) -> TransactionHash | ClaimReceipt`
- Validation: backend verifies claimant matches `employee` and computes claimable amount via canonical formula (see `contract/math.md`).

4. CancelStream

- Purpose: cancel a stream (partial refunds based on elapsed time and previously withdrawn amounts).
- Signature:
  - `CancelStream(streamId: StreamId, requester: PublicKey, cancelTimestamp: Timestamp) -> TransactionHash`
- Notes: cancellation rules are policy-defined (who may cancel, notice periods, admin roles).

5. PauseStream / ResumeStream

- Purpose: temporarily pause or resume accrual.
- Signature:
  - `PauseStream(streamId: StreamId, by: PublicKey, at: Timestamp) -> Void`
  - `ResumeStream(streamId: StreamId, by: PublicKey, at: Timestamp) -> Void`
- Side-effects: adjust effective accrual windows; indexer and math must account for paused intervals.

6. GetStream / ListStreams

- Read APIs used by frontend and indexer.
- `GetStream(streamId) -> StreamRecord` and `ListStreams(filter...) -> StreamRecord[]`.

---

## Events

- `StreamCreated(streamId, employer, employee, asset, rate, start, end, total)`
- `StreamFunded(streamId, txHash, amount)`
- `Claimed(streamId, claimant, amount, txHash)`
- `StreamCancelled(streamId, by, timestamp, refunded)`
- `StreamPaused(streamId, by, at)`
- `StreamResumed(streamId, by, at)`

---

## Example Stellar transaction flow (high-level)

- 1. Worker builds a payment or signed transaction to transfer `total` tokens from `employer` (or platform custody) to a distribution account or to pre-authorize claimable payouts.
- 2. Transaction includes memo or manage-data entries linking to `streamId` for indexer correlation.
- 3. Indexer watches Horizon for the transaction, links it to the `Stream` record, and updates `onChainBalance`.

## Notes

- Keep `contract/spec.md` in strict sync with implementation and recorded examples of actual Stellar transactions used in production or testnet.
