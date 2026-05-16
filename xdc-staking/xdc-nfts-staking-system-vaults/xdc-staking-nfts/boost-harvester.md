# Boost Harvester (technical)

[`XdcNftBoostHarvester`](../contract-addresses.md) is the small, non-upgradeable contract that funds the XDC NFT vault's Synthetix-style boost accumulator. It exists because the underlying [`PrimeStakedXDC_V3`](../contract-addresses.md) vault is non-upgradeable — the boost stream had to live in an external pump rather than being routed inside the V3 vault itself.

{% hint style="info" %}
Live address: [`0x3bEdb37FC873F64BEeFCA551b3A836e59fc18DeA`](https://xdcscan.com/address/0x3bEdb37FC873F64BEeFCA551b3A836e59fc18DeA). The harvester holds the **only** address granted `FEE_ROUTER_ROLE` on the NFT vault — i.e. it's the only contract allowed to call `notifyBoost`. Arbitrary XDC sends to the NFT vault cannot corrupt boost accounting.
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
| **Direct push** | `feed(uint256 amount)` payable | Treasury wants to push native XDC straight into the boost stream — fastest, no queue involved. |
| **NAV harvest** | `harvest(uint256 sharesToRedeem)` | The treasury seeded the harvester with psXDC v3 shares as principal; redeeming those shares captures the NAV gain over time. The XDC may arrive instantly (buffer covers it) or via the queue. |
| **Drain pending** | `forwardPending()` | Anyone can call this. Any native XDC sitting on the harvester (e.g. from a queued harvest that just settled) is forwarded into `notifyBoost`. |

---

## How the boost reaches NFT holders

When `notifyBoost(x)` runs on the NFT vault:

1. The vault calls `psXDC_v3.depositNative{value: x}(x, vault)` and receives `sharesMinted` of psXDC v3.
2. `rewardPerWeightStored += sharesMinted * 1e18 / totalWeight`.
3. Every staked NFT's pending boost immediately reflects the new value, proportional to the NFT's weight: `earned = info.shares * (rewardPerWeightStored - info.rewardIndex) * weight / 1e18`.
4. `notifyBoost` reverts if `totalWeight == 0` — pushing boost into an empty vault is a no-op so the value can't be wasted.

The Synthetix-style accumulator means **timing doesn't matter** as long as your NFT was staked when the push happened. You can claim now, later, or never — the value stays attributed to you.

---

## Why an external harvester

The original idea was to fund boost from psXDC v3's own NAV. That would have required adding a "fee skim" feature to the V3 vault. But the V3 vault is deliberately **non-upgradeable** (regular constructor, no proxy) so there is no way to change its logic after deployment. The harvester sidesteps this:

- Treasury seeds the harvester with psXDC v3 shares (or directly with XDC).
- When NAV has grown, the harvester redeems a portion via `redeemWithQueue` (going through the same instant-vs-queued path every user sees) and the resulting XDC funds the boost.
- The V3 vault itself never needs to know about boost; the harvester is the chokepoint.

The trade-off is that the harvester needs to be funded by an operator. The cadence is an operational choice — typically a weekly batch is cheapest gas-wise, daily is the friendliest UX. Either way, every `notifyBoost` emits a public event indexed by the subgraph so the UI can derive a trailing 30-day boost APR.

---

## Roles & safety

| Role | Holder | Why |
| --- | --- | --- |
| `DEFAULT_ADMIN_ROLE` (vault) | Protocol multisig | Master switch — can grant/revoke other roles |
| `FEE_ROUTER_ROLE` (vault) | `XdcNftBoostHarvester` | The only address allowed to call `notifyBoost` |
| `PAUSER_ROLE` (harvester) | Protocol multisig | Emergency stop |

The NFT vault deliberately has **no `receive()` function** — there is no way to "donate" XDC into the boost accumulator outside `notifyBoost`. This means random XDC sent to the vault cannot corrupt the accounting; only the harvester's `notifyBoost` calls move `rewardPerWeightStored`.

---

## What integrators can read

The harvester is a no-secret contract — every operation is on-chain and emits events. Useful read paths:

- **`BoostNotified(uint256 amountIn, uint256 sharesMinted, uint256 rewardPerWeightStored, uint256 totalWeight)`** on the NFT vault — each push.
- **`BoostFed` / `BoostHarvested` / `BoostForwarded`** on the harvester (or equivalent) — operational events.
- **`earned(tokenId)` view on the NFT vault** — pending boost for a specific NFT.

The subgraph at [`xdc-nft-v3-indexer`](https://github.com/PrimeNumbersLabs/xdc-nft-v3-indexer) provides aggregated entities including `BoostNotification`, per-NFT boost stats, and the `DailyProtocolSnapshot` series used by the UI.

---

## Pause behaviour

When the NFT vault is paused (`PAUSER_ROLE`), `stake` / `withdraw` / `claim` revert, but **`notifyBoost` continues to work** — this is intentional: boost flow keeps accruing even during an emergency pause, so users don't lose value during ops windows.

When the harvester itself is paused, no new pushes happen but pending value on the NFT vault is unaffected.

→ [Reward Model: Base NAV + Boost](xdc-nft-staking-reward-system.md) → [Staking Mechanics](xdc-staking-nfts-mechanics.md) → [Smart Contract Reference](smart-contract-functions.md)
