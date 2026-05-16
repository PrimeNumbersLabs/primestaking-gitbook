# Position & Rewards History

V3 does not have a "rewards claim" history in the V2 sense — rewards are baked into the share price and don't need to be claimed. Instead the app shows you the **history of your position**: every stake / migrate / withdraw event, and the exchange-rate trajectory over time so you can see your accrued value.

---

## How to View

1. Go to [primestaking.xyz/xdc-liquid-staking](https://primestaking.xyz/xdc-liquid-staking/overview).
2. Open the **My Positions** section. You'll see:
   - Your current psXDC share balance.
   - The current XDC value of that balance at the live exchange rate.
   - The cumulative XDC you've earned since your first stake (computed as `convertToAssets(balance) − netDeposits`).
3. Open the **Activity Log** for the per-transaction history: stakes, deposits, queued withdrawals, claims, and migrations. Each row links to the underlying transaction on XDCScan.

<figure><img src="../../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

---

## NFT boost history

If you also hold XDC NFTs, the boost slice (which **is** claimed) appears separately:

- The NFT detail page shows the pending boost amount.
- The Activity Log lists every `ClaimEvent`, `BoostNotification`, and weight-changing action (stake, lock, merge, withdraw).

→ [Understanding Share Price](how-to-claim-rewards.md) → [Reward Model: Base NAV + Boost](../../xdc-staking-nfts/xdc-nft-staking-reward-system.md)
