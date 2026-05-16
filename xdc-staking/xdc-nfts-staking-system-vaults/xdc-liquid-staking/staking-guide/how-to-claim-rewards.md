# Understanding Share Price

{% hint style="info" %}
There is **no "Claim Rewards" button** for XDC Liquid Staking in V3. Rewards accrue automatically through the psXDC share price — your shares simply become worth more XDC over time. This page explains why and how to read the value of your position.
{% endhint %}

If you are looking for the **NFT boost** claim flow (a separate XDC stream layered on top of the share), see [Reward Model: Base NAV + Boost](../../xdc-staking-nfts/xdc-nft-staking-reward-system.md).

If you are looking for the legacy V2 manual claim flow, see [Legacy (V2 — historical)](../../../../legacy/README.md).

---

## How rewards accrue in V3

The V3 vault is an ERC-4626 share contract. When validator rewards flow in, the vault's `totalAssets` increases but the total psXDC share supply does not, so the exchange rate `totalAssets / totalShares` goes up.

```
Day 0:   1,000 XDC deposited  →  1,000 psXDC minted  (rate = 1.000)
Day 30:  validator rewards flow in  →  totalAssets increases  →  rate = 1.0037
Day 365: cumulative rewards ≈ 4.5%  →  rate = 1.045

Burn 1,000 psXDC on day 365  →  receive 1,045 XDC
```

Your psXDC balance never changes from rewards. The **value** of your psXDC changes — exactly the same model Aave aTokens, Compound cTokens, and Lido wstETH use.

---

## Where to see your accrued value

The app surfaces this in two places:

- **Overview / Lite dashboard:** shows your psXDC share balance, its current XDC value at the live exchange rate, and the implied amount earned since you staked.
- **Position & Rewards History:** shows the exchange-rate timeline and your individual stake/withdraw events.

You can also read it directly from the vault contract:

```
sharesYouHold = balanceOf(yourAddress)
xdcYouCouldRedeemNow = convertToAssets(sharesYouHold)
```

The difference between `xdcYouCouldRedeemNow` and the XDC you originally deposited is your accrued yield. Nothing to claim — it is already inside the share.

---

## Realizing your rewards

Because rewards are baked into the share, you realize them whenever you do **any** of the following:

- **Redeem** psXDC for XDC — you receive XDC at the current (higher) exchange rate.
- **Sell** psXDC on a DEX — the market price reflects the appreciating NAV.
- **Transfer** psXDC to another wallet — the recipient inherits the appreciated share and any future appreciation.

There is no "leave rewards on the table" risk — rewards belong to whoever holds the share at the moment of redemption.

---

## What if I'm using my psXDC inside an XDC NFT?

The base NAV continues to accrue while your psXDC is staked inside an NFT — when you withdraw shares from the NFT, you get back the same share count, but each share is worth more XDC than when you deposited.

The **boost slice** is the additional XDC the NFT vault distributes via the `XdcNftBoostHarvester`. That stream **does** have a claim button — it accumulates inside the NFT and you collect it from the NFT detail page (or it settles automatically on any state-changing NFT action).

→ [Reward Model: Base NAV + Boost](../../xdc-staking-nfts/xdc-nft-staking-reward-system.md) → [Position & Rewards History](rewards-history.md)
