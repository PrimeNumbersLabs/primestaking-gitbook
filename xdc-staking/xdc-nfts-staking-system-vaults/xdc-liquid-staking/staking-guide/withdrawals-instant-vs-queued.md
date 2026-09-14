# Withdrawals: Instant vs Queued

Every withdrawal on V3 goes through `redeemWithQueue` (or `withdrawWithQueue`). The vault automatically picks the **fastest path your balance and the buffer allow**: no admin approval, no per-user setting to flip.

{% hint style="info" %}
**During the V3.1 collateral transition** (masternodes moving into the vault roughly weekly), instant withdrawals are served from the vault's free liquidity, and the FIFO queue is backed by dedicated team funding that is ring-fenced for queued requests. New stakers' deposits are never trapped, and queued users are paid as each liquidity tranche arrives. Once the transition completes, the standard buffer model below applies in full.
{% endhint %}

{% hint style="warning" %}
**If your request is queued right now:** several masternodes are unstaking at the XDC validator contract specifically to cover queued withdrawals. The network enforces its own unbonding period (`candidateWithdrawDelay`, ~35 days from resignation under typical block times); as that XDC is routed into the queue budget, requests are paid out in FIFO order (a request larger than the budget available at that moment is passed over and retried on the next pass, so smaller requests behind it can settle earlier). Your psXDC stays escrowed at its full share value while you wait and keeps following NAV — no action is needed from you, and you can cancel at any time to get your psXDC back.
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
2. For each request, values the escrowed shares at the **current** exchange rate (`previewRedeem(shares)`) and compares that against the **queue budget** (`queueBackingBudget`) — the XDC ring-fenced for queued requests. The vault's free liquidity that serves instant redemptions is *not* used for the queue.
3. If the budget covers the request, it burns the escrowed shares and sends XDC to the original receiver. If it does not, the request is **passed over, not cancelled** — the processor moves on and tries the next request, and the skipped one is retried on the next pass once more budget has arrived. In practice this means a small request can settle before a larger, older one when the budget is topped up in tranches.
4. If the receiver payout fails (e.g. a smart contract receiver that reverts on payment), the XDC is deferred into `pendingQueuedAssets[receiver]`. The user collects it later via `claimQueuedAssets`.

Auto-propose (the function that pushes new XDC into masternodes) is **blocked while there is any backlog**, so the protocol prioritizes outgoing user redemptions over locking up more XDC.

### What replenishes liquidity

Two separate pools of XDC matter here:

- The **instant buffer** (unencumbered vault balance) grows from **new user deposits** (any `stake` adds XDC) and any XDC returned from masternode resignations that is not earmarked for the queue. It serves `redeemWithQueue` calls that can settle immediately.
- The **queue budget** (`queueBackingBudget`) grows only from XDC explicitly routed to it: the team's liquidity tranches (sent by the migration-manager wallet, which the vault earmarks for queued requests) and, when a masternode is resigned through the vault, the returned principal up to the queue's unfunded amount. Instant withdrawals can never consume it, and deposits do not flow into it.

Masternode resignations return principal after the network's `candidateWithdrawDelay` (~35 days under typical block times), so for large backlogs that timeline is the upper bound on settlement.

### Queued requests keep following NAV until they are paid

Your request records the XDC value at enqueue time for the vault's accounting, but the **payout is recomputed when the request is processed**: the processor pays `previewRedeem(shares)` at the exchange rate of that moment. Reward distributions that land while you wait therefore *do* reach you — your escrowed shares keep appreciating exactly like unstaked psXDC. There is no reason to cancel and re-queue to capture a NAV increase; doing so only sends you to the back of the FIFO.

### The queue is public — and has an ETA

Every queued request is public on-chain data, and the app's **Queue Explorer** (My Positions page) shows the whole FIFO: each request's position, size, and owner wallet, plus the total queued. Your own requests are highlighted.

The explorer also shows an **expected completion date** and, when the team has published the unbonding schedule, a per-request **estimated payout date**. XDC masternodes return principal in **10M XDC lumps**, so the queue drains in steps: each request's estimate is the arrival date of the lump that covers its position.

**How the team dates each step**: the XDC validator contract enforces `candidateWithdrawDelay` = 1,296,000 blocks after a resignation. At the nominal 2-second block time that is 30–31 days, but real block times stretch it to roughly **35–38 days**, so published dates include a ~+5 day buffer on top of the nominal figure. These are good-faith estimates for planning, **not** on-chain guarantees. What the contract does guarantee: requests are processed in first-in-first-out order from the dedicated queue budget, a request the budget cannot yet cover is passed over (never cancelled) and retried on the next pass, escrowed shares are valued at the exchange rate of the moment they are paid, and you can cancel and recover your psXDC at any time.

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
| The owner picked which requests to honor | FIFO order is enforced on-chain; the only deviation is mechanical — a request the current budget cannot cover is passed over and retried, it is never cancelled or reprioritised by anyone |
| Withdrawals could be paused unilaterally by the admin | Auto-propose is blocked while the queue is non-empty, prioritizing exits over new validator locks |
| You had to wait the full validator-queue time even when liquidity was available | Instant when possible, queued only when the buffer is insufficient |
| Failed payouts could lose XDC | Failed payouts defer into `pendingQueuedAssets` and the user self-claims with `claimQueuedAssets` |

→ [Request Withdrawal (walkthrough)](request-withdrawal.md) → [Smart Contract Reference](../smart-contract-functions.md) → [V3 Architecture](../v3-architecture.md)
