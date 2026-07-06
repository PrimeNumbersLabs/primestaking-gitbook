# Staking Mechanics (V3)

XDC NFTs in V3 use a **weight-based** model rather than the V2 "monthly reward pool × multiplier" model. Your slice of the boost stream is proportional to your NFT's weight in the global accumulator, where weight is a function of staked shares, rarity, level, and lock status.

---

## The weight formula

```
weight(tokenId) = stakedShares × (rarityMultiplier + level + lockBonus)
```

| Term | What it represents |
| --- | --- |
| `stakedShares` | psXDC v3 shares currently deposited into this NFT |
| `rarityMultiplier` | Per-rarity constant set at mint (see [README](README.md#rarity-tiers)). Immutable on the V3 vault. |
| `level` | Increases as you stake more psXDC into the NFT, up to per-rarity caps |
| `lockBonus` | Additive integer applied while the NFT is locked, zeroed when unlocked |

All four terms are stored on-chain; the vault keeps `totalWeight` in sync so reward distribution stays consistent.

---

## The Synthetix-style accumulator

Boost rewards aren't distributed in monthly batches. Instead the vault uses a Synthetix-style accumulator that runs continuously:

```
on notifyBoost(x):
  sharesMinted = psXDC_v3.depositNative{value: x}(x, vault)
  rewardPerWeightStored += sharesMinted * 1e18 / totalWeight
  reverts if totalWeight == 0

earned(tokenId) = info.shares * (rewardPerWeightStored − info.rewardIndex) * weight / 1e18
```

What this means in practice:

- Whenever the harvester pushes XDC via `notifyBoost`, **every staked NFT's pending reward grows immediately** in proportion to its weight at that moment.
- `_settle(tokenId)` is called before any weight-changing operation (stake more, lock, unlock, merge, withdraw) so pending boost is captured against the **old** weight. You never overpay or underpay because of mid-stream changes.
- `notifyBoost` reverts if `totalWeight == 0`, since pushing boost into an empty vault is a no-op.

---

## Interface Actions

| Action | Function | What it does |
| --- | --- | --- |
| **Stake** | `stake(tokenId, shares)` | Deposit psXDC v3 shares against this NFT. Settles pending boost, updates weight, increments `level` if thresholds are crossed. |
| **Withdraw shares** | `withdraw(tokenId, shares)` | Remove psXDC shares without burning the NFT. Settles pending boost first. |
| **Lock** | `lock(tokenId, duration)` | Locks for a chosen tier duration (30/90/180/365 days), freezing that tier's `lockBonus` into the NFT and adding it to weight. Disables `withdraw`, `merge`, `burnAndRedeem` until the lock ends. |
| **Poke expired** | `pokeExpired(tokenIds)` | Permissionless keeper hook: retires the boost of any NFT whose lock has ended, so `totalWeight` stays accurate even for untouched NFTs. |
| **Merge** | `merge(tokenIdA, tokenIdB)` | Two same-rarity NFTs → one higher-rarity NFT. Burns originals and shares are released for restaking. |
| **Claim boost** | `claim(tokenId)` | Pays out earned boost in XDC. Can also unwrap to native XDC or keep as shares depending on the call. |
| **`burnAndRedeem`** | `burnAndRedeem(tokenId)` | Burns the NFT and returns the underlying psXDC v3 shares (or redeems them to XDC) in one transaction. |
| **Transfer** | ERC-721 `transferFrom` | NFT changes hands. The new owner inherits staked shares, weight, pending boost, and lock status. |

`mintAndStake` / `mintAndStakeLocked` are restricted to the migrator and are not directly callable by users.

---

## Locking

Locking an NFT does two things:

1. Sets `lockEnd = now + duration`. Until that timestamp passes, `withdraw`, `merge`, and `burnAndRedeem` revert.
2. Freezes the chosen tier's `lockBonus` into the NFT's weight, increasing its slice of every subsequent `notifyBoost`.

There are **four lock tiers** - 30, 90, 180 and 365 days - each granting a progressively larger boost. The boost you get is fixed at lock time (later tier-table changes don't affect an existing lock).

**Boost expiry.** Unlike earlier versions, the lock bonus **ends when the lock ends**. From the moment `lockEnd` passes, the NFT's effective weight drops back to its unlocked value, and the stale bonus is cleared on the next interaction (any user action, or a permissionless `pokeExpired` keeper call). Rewards already earned are never clawed back - the boost simply stops going forward. You can **re-lock** at any tier once a lock has expired.

The lock **does not change `stakedShares`**; the bonus is purely additive on the weight side.

Lock expiry is **preserved across migration** from V2; see [Locked NFTs & Legacy Diamond Bypass](locked-nft-migration.md).

---

## Caps and tuning

- `setLevelStakedNeeded` and `setLockBoost` can only be changed while `totalWeight == 0` (i.e. before any NFT is staked). Once the vault has live positions these setters revert, which prevents silent weight drift. Lock tiers are enabled/adjusted post-launch through the governance-gated `setLockBoostPostLaunch`, which only affects **future** locks - already-locked NFTs keep the boost they froze at lock time.
- `rarityMultiplier` has **no setter** on the V3 vault. Updating multipliers would require deploying a new vault and migrating.
- `setRarityMultiplier` does **not** exist on the live vault by design.
- The vault is paused with `PAUSER_ROLE`. Pausing halts stake/withdraw/claim; boost can still be received (intentional, to keep the stream flowing).

→ [Reward Model: Base NAV + Boost](xdc-nft-staking-reward-system.md) → [Smart Contract Reference](smart-contract-functions.md) → [Boost Harvester (technical)](boost-harvester.md)
