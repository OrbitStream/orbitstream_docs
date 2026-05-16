# Contract Math

This document defines the canonical formula for computing claimable balances for linear streams and provides worked examples and edge-case rules. Use these formulas in both off-chain indexing and on-chain verification where applicable.

## Canonical Linear Stream Formula

Given a stream with:

- `start` (timestamp)
- `end` (timestamp)
- `rate` (tokens per second)
- `total` (total tokens funded for the stream)
- `withdrawn` (amount already claimed)

The accrued amount at time $t$ is:

$$
accrued(t) = \begin{cases}
0 & t \le start \\
rate \times (\min(t, end) - start) & start < t < end \\
total & t \ge end
\end{cases}
$$

The claimable balance at time $t$ is:

$$
claimable(t) = \max(0, \min(accrued(t), total) - withdrawn)
$$

Notes:

- `rate` must satisfy $rate \times (end - start) = total$ in the fully-funded linear model. If `total` differs (partial funding), accrued is capped by `total`.
- When streams are paused, treat paused intervals by subtracting pause duration from the effective elapsed time used to compute `accrued(t)`.
- Cancellations reduce the future `total` available and should record `cancelTimestamp` at which accrual stops.

## Example 1 — Simple linear stream

- `start = 1000`, `end = 1900` (900 seconds), `total = 900`, so `rate = 1 token/sec`.
- At `t = 1300`, elapsed = 300s, `accrued = 300`, if `withdrawn = 0` then `claimable = 300`.

## Example 2 — Partial withdrawals

- Same stream as above, but `withdrawn = 250` and `t = 1300`.
- `accrued = 300`, `claimable = max(0, 300 - 250) = 50`.

## Example 3 — After end

- At `t = 2000`, `accrued = total = 900`, if `withdrawn = 800` then `claimable = 100`.

## Paused intervals

- If stream was paused from `t = 1200` to `t = 1400`, then at `t = 1500` effective elapsed = (1500 - 1000) - (1400 - 1200) = 300s, so `accrued = 300 * rate`.

## Implementation notes

- Use integer arithmetic for timestamps and canonical decimal handling for token amounts to avoid floating point drift.
- Persist pause/resume intervals and `withdrawn` totals in the off-chain index to compute claimable amounts idempotently.
