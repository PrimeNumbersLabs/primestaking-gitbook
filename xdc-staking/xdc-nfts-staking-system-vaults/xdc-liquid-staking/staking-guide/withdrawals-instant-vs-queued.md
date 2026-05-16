# Withdrawals: Instant vs Queued

Every withdrawal on V3 goes through `redeemWithQueue` (or `withdrawWithQueue`). The vault automatically picks the **fastest path your balance and the buffer allow** — no admin approval, no per-user setting to flip.

---

## The choice the vault makes for you

```
User calls redeemWithQueue(shares, receiver)
                        │
                        ▼
           ┌────────────────────────┐
           │  Does the buffer hold  │
   yes ◄───┤  enough XDC to cover   ├───► no
           │  shares × NAV?         │
           └────────────────────────┘
            │                          │
            ▼                          ▼
   Instant redeem in this tx   Escrow shares, enqueue FIFO
            │                          │
            ▼                          ▼
   Receive XDC immediately     Wait → call claimQueuedAssets later
```

The buffer is the percentage of total assets the vault keeps liquid (default 5%, tunable by the operations manager via `setBufferBps`). When `maxRedeem(you)` is greater than or equal to the shares you're redeeming, the path is instant. Otherwise the queue kicks in.

---

## Path A — Instant redeem

When liquidity is sufficient:

- Your psXDC shares are **burned immediately**.
- XDC is sent to your `receiver` in the same transaction.
- No queue entry is created.

Use cases: routine withdrawals while the vault has free liquidity, partner integrations expecting synchronous settlement.

---

## Path B — Queued redeem

When liquidity is constrained:

- Your psXDC shares are **escrowed inside the vault** (not burned).
- A `WithdrawalQueued(requestId, owner, shares)` event is emitted.
- The request enters the global FIFO queue, ordered by enqueue timestamp.

### How the queue is drained

Anyone can call `processWithdrawalQueue(maxRequests)`, which:

1. Walks the FIFO, oldest first.
2. For each request, checks the current liquid XDC against the request's share value at the **current** exchange rate.
3. If the vault can pay, it does — burns the escrowed shares, sends XDC to the original receiver.
4. If the receiver payout fails (e.g. a smart contract receiver that reverts on payment), the XDC is deferred into `pendingQueuedAssets[receiver]`. The user collects it later via `claimQueuedAssets`.

Auto-propose (the function that pushes new XDC into masternodes) is **blocked while there is any backlog**, so the protocol prioritizes outgoing user redemptions over locking up more XDC.

### What replenishes liquidity

The vault's liquid balance grows from:

- **New user deposits** (any `stake` adds XDC to the buffer).
- **Validator reward inflows** (rewards flow back into the vault and bump tracked assets).
- **Masternode resignation** — when a proposer resigns a masternode, the XDC returns to the vault after the network's `candidateWithdrawDelay` (~35 days under typical block times).

For very large withdrawals where the protocol doesn't already have a buffer + recent rewards sufficient to cover, the resignation timeline is the upper bound on settlement.

### Cancelling

You can call `cancelQueuedWithdrawal(requestId)` at any time before settlement. The escrowed psXDC shares are returned to your wallet — no XDC moves and nothing is lost.

### Self-claim with `claimQueuedAssets`

If you ever see a queued request that says "ready to claim" in the app, that means the queue processed your request but the XDC ended up in `pendingQueuedAssets` (e.g. your receiver bounced). Calling `claimQueuedAssets(receiver)` sweeps every XDC waiting for you into your wallet.

---

## How the app uses these paths

The PrimeStaking UI always calls `redeemWithQueue` — it never picks the path manually. Instead it shows you, before you sign, whether the transaction will:

- **"Withdraw complete"** — buffer is enough, this will settle now.
- **"Withdrawal queued — claim from My Positions when ready"** — buffer is not enough; the request will go into the FIFO.

In Lite Mode the **withdraw tab** uses the same logic. The **queue list** on the Withdraw and My Positions pages shows your active queued requests with cancel / claim controls.

---

## Why this design

| V2 behaviour | V3 behaviour |
| --- | --- |
| Every withdrawal required admin approval | No admin approval at any point |
| The owner picked which requests to honor | FIFO is enforced on-chain — first in, first out |
| Withdrawals could be paused unilaterally by the admin | Auto-propose is blocked while the queue is non-empty, prioritizing exits over new validator locks |
| You had to wait the full validator-queue time even when liquidity was available | Instant when possible, queued only when the buffer is insufficient |
| Failed payouts could lose XDC | Failed payouts defer into `pendingQueuedAssets` and the user self-claims with `claimQueuedAssets` |

→ [Request Withdrawal (walkthrough)](request-withdrawal.md) → [Smart Contract Reference](../smart-contract-functions.md) → [V3 Architecture](../v3-architecture.md)
