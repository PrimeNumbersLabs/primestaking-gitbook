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
- `_settle(tokenId)` is called before any weight-changing operation (stake more, lock, unlock, merge, withdraw) so pending boost is captured against the **old** weight — you never overpay or underpay because of mid-stream changes.
- `notifyBoost` reverts if `totalWeight == 0` — pushing boost into an empty vault is a no-op.

---

## Interface Actions

| Action | Function | What it does |
| --- | --- | --- |
| **Stake** | `stake(tokenId, shares)` | Deposit psXDC v3 shares against this NFT. Settles pending boost, updates weight, increments `level` if thresholds are crossed. |
| **Withdraw shares** | `withdraw(tokenId, shares)` | Remove psXDC shares without burning the NFT. Settles pending boost first. |
| **Lock** | `lock(tokenId, until)` | Sets `lockEnd`, adds `lockBonus` to weight. Disables `withdraw`, `merge`, `burnAndRedeem` until `lockEnd`. |
| **Unlock** | `unlock(tokenId)` | Once `lockEnd` has passed, removes `lockBonus` from weight. |
| **Merge** | `merge(tokenIdA, tokenIdB)` | Two same-rarity NFTs → one higher-rarity NFT. Burns originals and shares are released for restaking. |
| **Claim boost** | `claim(tokenId)` | Pays out earned boost in XDC. Can also unwrap to native XDC or keep as shares depending on the call. |
| **`burnAndRedeem`** | `burnAndRedeem(tokenId)` | Burns the NFT and returns the underlying psXDC v3 shares (or redeems them to XDC) in one transaction. |
| **Transfer** | ERC-721 `transferFrom` | NFT changes hands. The new owner inherits staked shares, weight, pending boost, and lock status. |

`mintAndStake` / `mintAndStakeLocked` are restricted to the migrator and are not directly callable by users.

---

## Locking

Locking an NFT does two things:

1. Sets `lockEnd` to a future timestamp. Until that timestamp passes, `withdraw`, `merge`, and `burnAndRedeem` revert.
2. Adds the configured `lockBonus` to the NFT's weight, increasing its slice of every subsequent `notifyBoost`.

The lock toggle **does not change `stakedShares`** — the bonus is purely additive on the weight side. When the lock expires you can `unlock` to remove the bonus and regain full mobility, or leave it locked to keep earning the boost slice.

Lock expiry is **preserved across migration** from V2 — see [Locked NFTs & Legacy Diamond Bypass](locked-nft-migration.md).

---

## Caps and tuning

- `setLevelStakedNeeded` and `setLockBoost` can only be changed while `totalWeight == 0` (i.e. before any NFT is staked). Once the vault has live positions these setters revert — this prevents silent weight drift.
- `rarityMultiplier` has **no setter** on the V3 vault. Updating multipliers would require deploying a new vault and migrating.
- `setRarityMultiplier` does **not** exist on the live vault by design.
- The vault is paused with `PAUSER_ROLE`. Pausing halts stake/withdraw/claim; boost can still be received (intentional — keep the stream flowing).

→ [Reward Model: Base NAV + Boost](xdc-nft-staking-reward-system.md) → [Smart Contract Reference](smart-contract-functions.md) → [Boost Harvester (technical)](boost-harvester.md)
