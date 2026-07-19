# Withdrawals: Instant vs Queued

Every withdrawal on V3 goes through `redeemWithQueue` (or `withdrawWithQueue`). The vault automatically picks the **fastest path your balance and the buffer allow**: no admin approval, no per-user setting to flip.

{% hint style="info" %}
**During the V3.1 collateral transition** (masternodes moving into the vault roughly weekly), instant withdrawals are served from the vault's free liquidity, and the FIFO queue is backed by dedicated team funding that is ring-fenced for queued requests. New stakers' deposits are never trapped, and queued users are paid as each liquidity tranche arrives. Once the transition completes, the standard buffer model below applies in full.
{% endhint %}

{% hint style="warning" %}
**If your request is queued right now:** several masternodes are unstaking at the XDC validator contract specifically to cover every queued withdrawal. The network enforces its own unbonding period (`candidateWithdrawDelay`, ~35 days from resignation under typical block times); as that XDC lands in the vault, the FIFO pays requests out **in order** and your request becomes claimable in the app. Every queued request is fully backed on-chain — no action is needed from you while you wait, and you can cancel at any time to get your psXDC back.
{% endhint %}

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

"Enough XDC" means the vault's **unencumbered liquidity**: its native balance minus what is already earmarked for the queue and for failed payouts (V3.2 removed the old percentage-based `bufferBps` buffer). When `maxRedeem(you)` covers the shares you're redeeming, the path is instant; otherwise the queue kicks in. Practically: while a queue backlog exists, most free liquidity is earmarked, so expect the queued path during those periods.

---

## Path A: Instant redeem

When liquidity is sufficient:

- Your psXDC shares are **burned immediately**.
- XDC is sent to your `receiver` in the same transaction.
- No queue entry is created.

Use cases: routine withdrawals while the vault has free liquidity, partner integrations expecting synchronous settlement.

---

## Path B: Queued redeem

When liquidity is constrained:

- Your psXDC shares are **escrowed inside the vault** (not burned).
- A `WithdrawalQueued(requestId, owner, shares)` event is emitted.
- The request enters the global FIFO queue, ordered by enqueue timestamp.

### How the queue is drained

Anyone can call `processWithdrawalQueue(maxRequests)`, which:

1. Walks the FIFO, oldest first.
2. For each request, checks the current liquid XDC against the request's share value at the **current** exchange rate.
3. If the vault can pay, it does: it burns the escrowed shares and sends XDC to the original receiver.
4. If the receiver payout fails (e.g. a smart contract receiver that reverts on payment), the XDC is deferred into `pendingQueuedAssets[receiver]`. The user collects it later via `claimQueuedAssets`.

Auto-propose (the function that pushes new XDC into masternodes) is **blocked while there is any backlog**, so the protocol prioritizes outgoing user redemptions over locking up more XDC.

### What replenishes liquidity

The vault's liquid balance grows from:

- **New user deposits** (any `stake` adds XDC to the buffer).
- **Validator reward inflows** (rewards flow back into the vault and bump tracked assets).
- **Masternode resignation**: when a proposer resigns a masternode, the XDC returns to the vault after the network's `candidateWithdrawDelay` (~35 days under typical block times).
- **Collateral-transition funding** (V3.1): while the legacy masternode fleet is being moved over, the team injects liquidity tranches on a rolling schedule; funding earmarked for the withdrawal queue is reserved for queued requests and cannot be consumed by instant withdrawals.

For very large withdrawals where the protocol doesn't already have a buffer + recent rewards sufficient to cover, the resignation timeline is the upper bound on settlement.

### Queued amounts are fixed — they don't earn while waiting

The XDC amount of a queued request is locked in at the exchange rate of the moment you queued (`previewRedeem` at enqueue time). Reward distributions that land while you wait do **not** increase your payout — from the vault's perspective your exit price is already settled; only the timing of payment is pending. This is standard exit-queue design: fixing the liability at request time keeps the vault's solvency accounting exact.

Because `cancelQueuedWithdrawal` returns your **shares** (not the fixed amount), cancelling after a rate increase and re-queueing captures the appreciation — but it sends you to the back of the FIFO. At ~5.5% APY that trade-off is roughly 0.45% per month of queue time, so it is rarely worth it unless you were far from the head anyway.

### The queue is public — and has an ETA

Every queued request is public on-chain data, and the app's **Queue Explorer** (My Positions page) shows the whole FIFO: each request's position, size, and owner wallet, plus the total queued. Your own requests are highlighted.

The explorer also shows an **expected completion date** and, when the team has published the unbonding schedule, a per-request **estimated payout date**. XDC masternodes return principal in **10M XDC lumps**, so the queue drains in steps: each request's estimate is the arrival date of the lump that covers its position.

**How the team dates each step**: the XDC validator contract enforces `candidateWithdrawDelay` = 1,296,000 blocks after a resignation. At the nominal 2-second block time that is 30–31 days, but real block times stretch it to roughly **35–38 days**, so published dates include a ~+5 day buffer on top of the nominal figure. These are good-faith estimates for planning, **not** on-chain guarantees. What the contract does guarantee: requests are paid strictly first-in-first-out as XDC arrives, and every request is fully backed.

### Cancelling

You can call `cancelQueuedWithdrawal(requestId)` at any time before settlement. The escrowed psXDC shares are returned to your wallet; no XDC moves and nothing is lost.

### Self-claim with `claimQueuedAssets`

If you ever see a queued request that says "ready to claim" in the app, that means the queue processed your request but the XDC ended up in `pendingQueuedAssets` (e.g. your receiver bounced). Calling `claimQueuedAssets(receiver)` sweeps every XDC waiting for you into your wallet.

---

## How the app uses these paths

The PrimeStaking UI always calls `redeemWithQueue`; it never picks the path manually. Instead it shows you, before you sign, whether the transaction will:

- **"Withdraw complete"**: buffer is enough, this will settle now.
- **"Withdrawal queued, claim from My Positions when ready"**: buffer is not enough; the request will go into the FIFO.

In Lite Mode the **withdraw tab** uses the same logic. The **queue list** on the Withdraw and My Positions pages shows your active queued requests with cancel / claim controls.

---

## Why this design

| V2 behaviour | V3 behaviour |
| --- | --- |
| Every withdrawal required admin approval | No admin approval at any point |
| The owner picked which requests to honor | FIFO is enforced on-chain: first in, first out |
| Withdrawals could be paused unilaterally by the admin | Auto-propose is blocked while the queue is non-empty, prioritizing exits over new validator locks |
| You had to wait the full validator-queue time even when liquidity was available | Instant when possible, queued only when the buffer is insufficient |
| Failed payouts could lose XDC | Failed payouts defer into `pendingQueuedAssets` and the user self-claims with `claimQueuedAssets` |

→ [Request Withdrawal (walkthrough)](request-withdrawal.md) → [Smart Contract Reference](../smart-contract-functions.md) → [V3 Architecture](../v3-architecture.md)
