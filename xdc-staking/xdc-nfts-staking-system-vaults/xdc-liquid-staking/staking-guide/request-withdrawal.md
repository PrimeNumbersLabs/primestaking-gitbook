# Request Withdrawal

V3 withdrawals are self-service: there is no admin approval step and no fixed queue time. Every withdrawal goes through `redeemWithQueue`, which automatically picks the fastest path your balance and the vault's liquidity allow.

{% hint style="info" %}
For the full breakdown of how the instant vs queued paths choose themselves, see [Withdrawals: Instant vs Queued](withdrawals-instant-vs-queued.md).
{% endhint %}

---

## Steps

1. Go to [primestaking.xyz/xdc-liquid-staking](https://primestaking.xyz/xdc-liquid-staking/overview).
2. Open the **Withdraw** section (or use the **Lite Mode** withdraw tab).
3. Enter the amount of **psXDC shares** you want to redeem. The app shows you, in real time:
   - The XDC you will receive based on the current exchange rate.
   - Whether the withdrawal will settle **instantly** or **enter the queue**, based on the vault's liquid buffer.
4. Click **Withdraw** and confirm the transaction. Under the hood the app calls `redeemWithQueue(shares, receiver)`.

---

## What happens next

### If the vault has enough buffer liquidity

The redemption settles in the **same transaction**. You receive XDC immediately. No queue entry is created, nothing else for you to do.

### If buffer liquidity is constrained

Your psXDC shares are escrowed inside the vault and a request is added to the FIFO queue. The withdrawal **does not have a fixed time** — it is settled as soon as enough liquidity returns from:

- New user deposits,
- Validator reward inflows, or
- Masternode resignation principal returning to the vault after the XDC Network's `candidateWithdrawDelay` (~35 days under normal block times).

You can monitor the queue at any time from the **My Positions** page. When your request is processed, your XDC lands either directly in your wallet or in the vault's `pendingQueuedAssets` bucket. If it lands in `pendingQueuedAssets` (because the original payout failed for any reason), you collect it by calling `claimQueuedAssets`. The app exposes this as a **Claim** button on the queued withdrawal entry.

You can also cancel a queued request before it settles. The vault returns the escrowed psXDC shares to your wallet — no XDC moves.

---

## Why a queue exists at all

The vault keeps most of the XDC working in masternodes earning yield, and only holds a small percentage (the **buffer**, default 5%) liquid for instant redemptions. When demand to withdraw exceeds the buffer, the queue ensures that everyone is served fairly in FIFO order without forcing the protocol to disrupt active validators.

The queue is preferred over the previous "request and wait for admin approval" model because:

- No admin signature is needed at any point.
- You can cancel at any time and get your shares back.
- Settlement is automatic — anyone can call `processWithdrawalQueue` to push the queue forward.

→ [Withdrawals: Instant vs Queued](withdrawals-instant-vs-queued.md) → [Smart Contract Reference](../smart-contract-functions.md)

<figure><img src="../../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>
