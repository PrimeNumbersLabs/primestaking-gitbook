# How Rewards Work

PrimeStaking V3 is a fully on-chain, ERC-4626 staking vault. Rewards are not paid through a manual `claim` flow — they are embedded directly in the **psXDC share price**. This page explains how the rate accrues and how the NFT boost layer sits on top.

---

## Two layers of yield

```
                base yield (~4.5%)               boost slice (up to ~1.5%)
                       │                                  │
                       ▼                                  ▼
   shares × NAV(t) appreciation     +     Synthetix-style accumulator
   (every psXDC holder gets this)         (only XDC NFT holders get this)
```

| Layer | Where it lives | Who earns it | How you receive it |
| --- | --- | --- | --- |
| **Base NAV** | `PrimeStakedXDC_V3` vault share price | Every psXDC holder | Automatic — share price rises as validator rewards accrue. No claim. |
| **NFT Boost** | `XdcNftStakingVault` Synthetix accumulator | Only NFTs that have psXDC shares staked inside them | Claimed from the NFT detail page in the app. |

---

## Base layer — share-price growth

When XDC validator rewards flow back into the vault, `totalAssets()` increases while the total share supply stays the same. The vault's exchange rate (`totalAssets / totalShares`) goes up, so every psXDC share becomes worth more XDC.

| Aspect | Detail |
| --- | --- |
| Target APY | ~4.5% |
| Mechanism | Exchange rate appreciation |
| When you receive it | Continuously — visible the next time the vault syncs reward inflows |
| User action required | None — there is no "claim rewards" button |

**Example.** Stake 1,000 XDC when the exchange rate is `1.00`. You receive 1,000 psXDC shares. Six months later the rate is `1.025`. You burn your 1,000 shares and the vault returns **1,025 XDC** — your 25 XDC of yield was inside the price the whole time.

This is the same model used by Aave aTokens, Compound cTokens, and Lido wstETH.

---

## NFT boost — additional slice for stakers

XDC NFTs stack a second yield on top of the base NAV. The protocol's [`XdcNftBoostHarvester`](../xdc-staking-nfts/boost-harvester.md) periodically pushes XDC into the NFT vault via `notifyBoost`. That XDC is converted to psXDC v3 shares inside the vault and parked as `boostReserve`. A Synthetix-style accumulator distributes the reserve to staked NFTs in proportion to their **weight**:

```
weight = stakedShares × (rarityMultiplier + level + lockBonus)
```

| Component | Effect on weight |
| --- | --- |
| `stakedShares` | More psXDC staked inside the NFT → bigger slice |
| `rarityMultiplier` | Plentiful (lowest) → Handcrafted (highest) — set at mint, immutable |
| `level` | Increases when the NFT levels up via merge |
| `lockBonus` | Added when the NFT is locked, zeroed when unlocked |

When you `claim` from an NFT, the accumulator settles your pending boost and pays it out in XDC.

| Aspect | Detail |
| --- | --- |
| Target APR band | ~0.25% (Plentiful unlocked) → ~1.5% (Handcrafted locked) |
| Mechanism | `notifyBoost(x)` increments `rewardPerWeightStored`; each NFT's `earned = info.shares × (rewardPerWeightStored − info.rewardIndex) × weight / 1e18` |
| When you receive it | When you call `claim` on the NFT |
| User action required | Yes — claim from the NFT detail page; or it settles automatically on any other state-changing action (stake more, lock, merge, withdraw) |

The boost cadence depends on how often the harvester feeds the accumulator — see [Boost Harvester (technical)](../xdc-staking-nfts/boost-harvester.md) for the full picture.

---

## Combined targets

| Position | Base NAV | + Boost slice | Target APY |
| --- | --- | --- | --- |
| Plain psXDC (no NFT) | ~4.5% | — | **~4.5%** |
| psXDC inside an unlocked NFT | ~4.5% | ~0.25% | **~4.75%** |
| psXDC inside a locked NFT | ~4.5% | up to ~1.5% | **up to ~6%** |

The exact boost slice depends on your NFT's rarity, level, and lock status relative to the rest of the vault — and on how much XDC the harvester has fed recently. The vault publishes `BoostNotified` events so the UI can show a trailing 30-day boost APR alongside the static targets.

---

## Why no claim button for liquid staking

The V2 product distributed rewards by having the owner periodically call `notifyRewardAmount` and required users to call `claim` to collect their share. V3 removes both:

- **No `notifyRewardAmount`** — validator rewards flow directly into the vault, so the exchange rate updates automatically.
- **No `claim`** — your reward is *already inside the share*. When you redeem the share, you receive both the principal and the accrued reward in one transaction.

The boost layer keeps its `claim` flow because it is a **separate** XDC stream layered on top of the share, not an internal accrual.

→ [Understanding Share Price](staking-guide/how-to-claim-rewards.md) → [V2 vs V3 — What Changed](v2-vs-v3.md)
