# Boost Harvester (technical)

[`XdcNftBoostHarvester`](../contract-addresses.md) is a small, non-upgradeable contract that can fund the XDC NFT vault's Synthetix-style boost accumulator with native XDC. It was designed as an external pump because the underlying psXDC vault is non-upgradeable. Since the **14 Sep 2026** NFT-vault upgrade the harvester is optional: the operations wallet holds `FEE_ROUTER_ROLE` directly and funds the boost with `notifyBoost` (XDC) or the new `notifyBoostShares` (psXDC shares, no XDC needed).

{% hint style="info" %}
Live address: [`0x6a319528111E5e50712Fd2D3d2db8323b119821D`](https://xdcscan.com/address/0x6a319528111E5e50712Fd2D3d2db8323b119821D). `FEE_ROUTER_ROLE` on the NFT vault is held by this harvester **and** by the operations wallet (`0x440c…113d`); nothing else can call `notifyBoost` / `notifyBoostShares`, and arbitrary XDC sends to the NFT vault cannot corrupt boost accounting. Note that this harvester instance is wired to the retired V3.1 psXDC token, so only its native-XDC lanes (`feed`, `forwardPending`) are usable — `harvest` / `claimAndForward` are not.
{% endhint %}

---

## What the harvester does

```
┌────────────────────────────────────────────────────────────────────────┐
│                     XDC sources (treasury / NAV)                        │
│                                                                         │
│   feed(amount) ────────► forwards native XDC directly to notifyBoost   │
│                                                                         │
│   harvest(sharesToRedeem) ──► redeemWithQueue on psXDC v3 vault ──►    │
│       …queued or instant XDC payout…                                    │
│       forwardPending() picks up the XDC and pushes notifyBoost          │
└────────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
            XdcNftStakingVault.notifyBoost(amount) payable
                  - converts XDC → psXDC v3 shares via depositNative
                  - bumps rewardPerWeightStored += sharesMinted * 1e18 / totalWeight
                  - reverts if totalWeight == 0
```

### Two funding paths

| Path | Function | When to use it |
| --- | --- | --- |
| **Direct push** | `feed(uint256 amount)` payable | Treasury wants to push native XDC straight into the boost stream. Fastest path, no queue involved. |
| **NAV harvest** | `harvest(uint256 sharesToRedeem)` | The treasury seeded the harvester with psXDC v3 shares as principal; redeeming those shares captures the NAV gain over time. The XDC may arrive instantly (buffer covers it) or via the queue. |
| **Drain pending** | `forwardPending()` | Anyone can call this. Any native XDC sitting on the harvester (e.g. from a queued harvest that just settled) is forwarded into `notifyBoost`. |

---

## How the boost reaches NFT holders

When `notifyBoost(x)` runs on the NFT vault:

1. The vault calls `psXDC_v3.depositNative{value: x}(x, vault)` and receives `sharesMinted` of psXDC v3.
2. `rewardPerWeightStored += sharesMinted * 1e18 / totalWeight`.
3. Every staked NFT's pending boost immediately reflects the new value, proportional to the NFT's weight: `earned = info.shares * (rewardPerWeightStored - info.rewardIndex) * weight / 1e18`.
4. `notifyBoost` reverts if `totalWeight == 0`; pushing boost into an empty vault is a no-op so the value can't be wasted.

The Synthetix-style accumulator means **timing doesn't matter** as long as your NFT was staked when the push happened. You can claim now, later, or never; the value stays attributed to you.

---

## Why an external harvester

The original idea was to fund boost from psXDC v3's own NAV. That would have required adding a "fee skim" feature to the V3 vault. But the V3 vault is deliberately **non-upgradeable** (regular constructor, no proxy) so there is no way to change its logic after deployment. The harvester sidesteps this:

- Treasury seeds the harvester with XDC (`feed` / `forwardPending`). The psXDC-principal lane (`depositPrincipal` → `harvest`) is unusable on the current instance because it points at the retired V3.1 token.
- In practice the boost is now paid directly by the operations wallet: `notifyBoostShares(shares)` pulls psXDC from it and credits every NFT in one transaction, without touching the psXDC vault's liquidity buffer.
- The psXDC vault itself never needs to know about boost; the NFT vault's `FEE_ROUTER_ROLE` is the chokepoint.

The trade-off is that the boost has to be funded by an operator, so it arrives in batches. History so far: no boost was pushed between the V3 launch (14 May 2026) and **14 Sep 2026**, when the full ~1.5% band for that period (250 + 85,850 psXDC ≈ 86,700 XDC) was distributed in one `notifyBoostShares` batch; from there the target cadence is monthly (~1.5% p.a. on the staked total). Every push emits a public `BoostNotified` event indexed by the subgraph, and the app derives the displayed boost APR from the boost paid since launch, annualised and capped at the marketed band.

---

## Roles & safety

| Role | Holder | Why |
| --- | --- | --- |
| `DEFAULT_ADMIN_ROLE` (vault) | Protocol multisig | Master switch; can grant/revoke other roles |
| `FEE_ROUTER_ROLE` (vault) | `XdcNftBoostHarvester` and the operations wallet `0x440c…113d` | The only addresses allowed to call `notifyBoost` / `notifyBoostShares` |
| `PAUSER_ROLE` (harvester) | Protocol multisig | Emergency stop |

The NFT vault deliberately has **no `receive()` function**, so there is no way to "donate" XDC into the boost accumulator outside `notifyBoost` / `notifyBoostShares`. This means random XDC or psXDC sent to the vault cannot corrupt the accounting; only `FEE_ROUTER_ROLE` calls move `rewardPerWeightStored`.

---

## What integrators can read

The harvester is a no-secret contract: every operation is on-chain and emits events. Useful read paths:

- **`BoostNotified(uint256 amountIn, uint256 sharesMinted, uint256 rewardPerWeightStored, uint256 totalWeight)`** on the NFT vault, emitted on each push.
- **`BoostFed` / `BoostHarvested` / `BoostForwarded`** on the harvester (or equivalent): operational events.
- **`earned(tokenId)` view on the NFT vault**: pending boost for a specific NFT.

The subgraph at [`xdc-nft-v3-indexer`](https://github.com/PrimeNumbersLabs/xdc-nft-v3-indexer) provides aggregated entities including `BoostNotification`, per-NFT boost stats, and the `DailyProtocolSnapshot` series used by the UI.

---

## Pause behaviour

When the NFT vault is paused (`PAUSER_ROLE`), `stake` / `withdraw` / `claim` revert, but **`notifyBoost` continues to work**. This is intentional: boost flow keeps accruing even during an emergency pause, so users don't lose value during ops windows.

When the harvester itself is paused, no new pushes happen but pending value on the NFT vault is unaffected.

→ [Reward Model: Base NAV + Boost](xdc-nft-staking-reward-system.md) → [Staking Mechanics](xdc-staking-nfts-mechanics.md) → [Smart Contract Reference](smart-contract-functions.md)
