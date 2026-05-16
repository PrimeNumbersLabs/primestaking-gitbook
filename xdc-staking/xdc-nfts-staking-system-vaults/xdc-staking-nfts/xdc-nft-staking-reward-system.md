# Reward Model: Base NAV + Boost

XDC NFTs in V3 earn from **two stacked sources**. There is no monthly XDC pool and no "PrimeFi ecosystem profit share" math — both layers are fully on-chain and continuously accruing.

---

## The two layers

```
                  base APY (~4.5%)            boost APY (up to ~1.5%)
                       │                              │
                       ▼                              ▼
   shares × NAV(t) appreciation     +     Synthetix accumulator slice
   (the psXDC v3 share price)              (rarityMult + level + lockBonus weighted)
```

### 1. Base NAV — psXDC v3 share-price appreciation

Every psXDC v3 share grows in value as validator rewards flow into the underlying vault. When your NFT holds `stakedShares` of psXDC, the **same share count** is returned on `withdraw`, but each share is worth more XDC than it was on `stake`. This layer requires **no action and no claim** — it's already inside the shares.

| Aspect | Detail |
| --- | --- |
| Target APY | ~4.5% |
| How it accrues | Via [`PrimeStakedXDC_V3`](../contract-addresses.md) share-price growth |
| When you realize it | When you `withdraw` shares from the NFT or `burnAndRedeem` |

### 2. Boost — Synthetix accumulator inside the NFT vault

The protocol's [`XdcNftBoostHarvester`](boost-harvester.md) periodically pushes XDC into the NFT vault via `notifyBoost`. The vault converts that XDC to psXDC v3 shares and credits the accumulator. Every staked NFT earns a slice proportional to its **weight**:

```
weight = stakedShares × (rarityMultiplier + level + lockBonus)
```

| Aspect | Detail |
| --- | --- |
| Target APR band | ~0.25% (Plentiful unlocked) → ~1.5% (Handcrafted locked) |
| How it accrues | `rewardPerWeightStored` increments on every `notifyBoost`; per-NFT `earned` is computed Synthetix-style |
| When you realize it | When you call `claim(tokenId)` from the NFT detail page (or automatically on any other state-changing action like stake/lock/merge/withdraw, which `_settle` first) |
| Reward asset | XDC (the harvester's payload is native XDC) |

---

## Combined target ranges

| Position | Base NAV (floor) | + Boost slice (when flowing) | Combined target |
| --- | --- | --- | --- |
| Plain psXDC, no NFT | ~4.5% | — | **~4.5%** |
| psXDC staked in an unlocked NFT | **~4.5%** | ~0.25% | **~4.5% → ~4.75%** |
| psXDC staked in a locked NFT | **~4.5%** | up to ~1.5% | **~4.5% → up to ~6%** |

{% hint style="info" %}
**The floor for every staked NFT is the base ~4.5%.** That layer is purely psXDC v3 share-price growth — it accrues automatically and does not depend on rarity, level, lock status, or harvester cadence. Even if `notifyBoost` hasn't been called in a while, you still earn the base. The boost slice is an **additional** stream on top.
{% endhint %}

Your individual boost APR depends on:

- Your NFT's `rarityMultiplier`, `level`, and whether it's locked.
- How much psXDC you have staked (more shares → more weight → more slice).
- How active the harvester's `notifyBoost` stream has been recently.

The UI surfaces a trailing 30-day boost APR alongside the static targets so you can see what the stream has actually paid.

---

## Why no monthly pool

The V2 NFT system distributed rewards in a monthly batch process driven off-chain. V3 replaces this with a **continuous, on-chain Synthetix accumulator** for three reasons:

- **Trust-minimized** — the boost rate is a function of harvester pushes, not a manually-set monthly figure.
- **Granular** — earnings update on every `notifyBoost`, not once per month.
- **Composable** — partner integrations can read `earned(tokenId)` directly on-chain at any time.

There is no longer a "10% of PrimeFi profits" framing; the harvester's funding model is described in [Boost Harvester (technical)](boost-harvester.md).

---

## Maximizing Your Rewards

| Strategy | Effect |
| --- | --- |
| Stake more psXDC shares | More `stakedShares` → linear increase in weight |
| Merge two same-rarity NFTs | Higher `rarityMultiplier` on the result |
| Level the NFT | Higher `level` term, additive to the multiplier |
| Lock the NFT | Adds `lockBonus` to the weight, but disables withdraw/merge/burnAndRedeem until expiry |
| Hold higher-rarity NFTs | Higher base `rarityMultiplier` = larger slice for the same staked shares |

→ [Staking Mechanics](xdc-staking-nfts-mechanics.md) → [Boost Harvester (technical)](boost-harvester.md) → [Smart Contract Reference](smart-contract-functions.md)
