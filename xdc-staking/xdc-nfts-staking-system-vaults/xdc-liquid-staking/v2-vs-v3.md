# V2 vs V3 — What Changed and Why

{% hint style="info" %}
The live vault is **`PrimeStakedXDC_V3_1`** (`0xa7FD…73e4`). V2 remains available for stragglers to migrate via [the migration bridge](staking-guide/migration.md).
{% endhint %}

psXDC has moved from its original custodial contract (V2) to a fully non-custodial, ERC-4626 vault architecture (V3). This page explains what changed, why, and what it means for users and partners.

---

## July 2026: the V3.1 redeploy

In July 2026 the V3 vault was redeployed as **V3.1** to restructure how masternode collateral moves into the vault. What users need to know:

- **Balances carried over automatically.** A snapshot captured every V3 holder's balance — including psXDC inside XDC NFTs, DEX pools, lending markets, and open limit orders — and the `V31AirdropDistributor` minted the same balances on V3.1. No user action was required.
- **Staking, withdrawals, and NFTs work exactly the same.** V3.1 keeps the full ERC-4626 share design, self-service withdrawals, and the FIFO queue.
- **Masternode collateral migrates progressively.** The XDC backing the vault is being moved from the legacy masternode fleet into the V3.1 vault on a rolling schedule (roughly one masternode per week). While this completes, the vault serves withdrawals from its on-hand liquidity plus dedicated team funding lanes; larger withdrawals may queue until the next liquidity tranche arrives. Share price (NAV) is protected during this phase — it cannot be written down by the transition mechanics.
- **The old V3 token is retired.** Its bridge was cut to a dead address; the token has no remaining function.

Everywhere else in this page, "V3" refers to the live V3.1 architecture — the design is identical.

---

## Why Migrate

V2 was the first generation of psXDC. It works, but its custodial design creates trust dependencies that limit adoption — particularly as collateral in DeFi lending markets. The core issues:

- **Withdrawals require admin approval** — a user requests to withdraw, then the owner must manually approve each request before XDC is released
- **Custodial validator staking** — the admin withdrew XDC from the contract and staked it into validators; staking happened on-chain but was entirely admin-controlled, so users had to trust the team to manage their funds
- **Owner can mint and withdraw** — the contract has `mint()` and `ownerWithdraw()` functions that give the owner broad control over user funds
- **Rewards are manual** — the owner must call `notifyRewardAmount()` to distribute rewards

These properties make V2 unsuitable for institutional use or as DeFi collateral, because a single admin key can unilaterally move funds.

V3 eliminates all of these trust assumptions.

---

## Architecture Comparison

| Aspect | V2 (Custodial) | V3 (Non-Custodial) |
| --- | --- | --- |
| **Standard** | Custom staking + APY rewards | ERC-4626 tokenized vault |
| **Token model** | 1:1 fixed ratio (1 psXDC = 1 XDC) | Share-based exchange rate (increases over time) |
| **Reward distribution** | Manual — owner calls `notifyRewardAmount()` | Automatic — exchange rate rises as validator rewards flow in |
| **Reward claiming** | Users must manually claim accrued rewards | No claiming — rewards are embedded in share price |
| **XDC utilization** | Custodial — admin withdrew XDC and staked with validators manually | Non-custodial — vault stakes directly with masternodes via smart contract, fully verifiable |
| **Masternode integration** | Custodial — admin managed validators manually | Direct non-custodial integration with XDC validator contract |
| **Upgradeability** | UUPS (owner-controlled upgrades) | **None — `PrimeStakedXDC_V3_1` is non-upgradeable.** Deployed with a regular constructor, no proxy. The vault logic can never be modified. |

---

## Withdrawal Comparison

| Aspect | V2 | V3 |
| --- | --- | --- |
| **User action** | Call `requestWithdraw()` | Call `redeem()` or `redeemWithQueue()` |
| **Admin approval** | Required — owner must call `approveWithdrawRequest()` | Not required — self-service |
| **Instant withdrawal** | Not available | Yes — if sufficient liquid XDC exists in the buffer |
| **Queue** | Manual admin-managed | Automatic FIFO queue — processes as liquidity returns |
| **Queue cancellation** | Not available | Users can cancel pending requests at any time |
| **DEX exit** | Available (same in both versions) | Available (same in both versions) |

**V2 flow:** Request → Wait for admin approval → Withdraw

**V3 flow:** Redeem → Receive XDC instantly (or enter automatic queue if liquidity is low)

---

## Admin Privilege Comparison

This is the most significant change. V2 gives the owner sweeping powers. V3 eliminates them.

| Capability | V2 | V3 |
| --- | --- | --- |
| **Mint new tokens** | Yes — `mint(address, amount)` | Disabled — `mint()` reverts |
| **Withdraw contract funds** | Yes — `ownerWithdraw(amount)` | No such function exists |
| **Set reward rate** | Yes — `setRewardApy()` | No manual rate — exchange rate is automatic |
| **Approve individual withdrawals** | Yes — `approveWithdrawRequest()` | Not needed — withdrawals are self-service |
| **Transfer ownership** | Instant — `transferOwnership()` | Time-locked — schedule → wait → execute |
| **Change roles** | N/A (single owner) | Time-locked — all role changes require delay |
| **Pause/unpause** | Owner | Admin role |

---

## Reward Model Comparison

| Aspect | V2 | V3 |
| --- | --- | --- |
| **How rewards enter** | Owner calls `notifyRewardAmount(reward)` with XDC | Validator rewards flow directly into the vault |
| **Distribution logic** | Time-based APY: `rewardRate * elapsed / totalStaked` | Asset-based: exchange rate = `totalAssets / totalShares` |
| **User experience** | Claim rewards manually from Rewards tab | No claim — share value increases automatically |
| **Transparency** | Owner-controlled reward injection | Validator rewards verifiable on-chain via masternode contract |
| **Risk** | Owner sets APY and reward duration manually | Exchange rate driven by actual validator performance |
| **NFT boost stream** | N/A | `XdcNftBoostHarvester.notifyBoost` feeds a Synthetix-style accumulator inside the NFT vault — claimable separately |

---

## Security Comparison

| Feature | V2 | V3 |
| --- | --- | --- |
| **Audit** | QuillAudits | QuillAudits + Nethermind Security |
| **Reentrancy protection** | Yes | Yes (OpenZeppelin ReentrancyGuard) |
| **Single point of failure** | Yes — single owner key | No — 5 roles + time-locks |
| **Upgradeability** | UUPS proxy | **None — non-upgradeable** |
| **Governance delay** | None | 1-day minimum (configurable, max 30 days) |
| **Loss caps** | None | Configurable per-report + per-day caps, set via delayed governance |
| **Principal tracking** | None | Per-operator and global tracking |
| **Admin can drain funds** | Technically yes (`ownerWithdraw`, `mint`) | No — no such functions exist |
| **No principal-stake slashing** | Yes (XDC model) | Yes (XDC model) |

---

## Feature-by-Feature Summary

| Feature | V2 | V3 |
| --- | --- | --- |
| Stake XDC | Yes | Yes |
| Receive psXDC | Yes (1:1) | Yes (at exchange rate) |
| Self-service withdrawal | No | Yes |
| Withdrawal queue | Manual | Automatic (FIFO) |
| Cancel queued withdrawal | No | Yes |
| Masternode staking | Off-chain | On-chain |
| Auto-propose masternodes | No | Yes |
| Resign masternodes | N/A | Yes (with cooldown) |
| Share-based pricing | No | Yes |
| Liquidity buffer | No | Yes (configurable, default 5%) |
| Time-locked governance | No | Yes (all sensitive changes) |
| Role-based access control | No (single owner) | Yes (5 distinct roles) |
| Validator loss reporting | No | Yes (capped, operator-aware) |
| ERC-4626 compatible | No | Yes |
| DeFi composable (collateral) | Limited | Full |
| Migration bridge | N/A | Dedicated contract with slippage protection |

---

## What This Means for Users

| | Before (V2) | After (V3) |
| --- | --- | --- |
| **Deposit** | Same — stake XDC, get psXDC | Same — but psXDC is now a yield-bearing vault share |
| **Rewards** | Claim manually from Rewards tab | Automatic — no action needed, share value grows |
| **Withdraw** | Request and wait for admin approval | Self-service — instant if liquidity available, queued otherwise |
| **Trust** | Trust the admin not to misuse `mint()` or `ownerWithdraw()` | Trust the code — no admin can move your funds, and the contract itself cannot be upgraded |
| **DeFi use** | Hold, trade, or stake in NFTs | Same + usable as collateral in lending protocols |
| **Migration** | — | One-time: approve + migrate via the [V3 migration bridge](staking-guide/migration.md) |

---

## What This Means for Partners

- **psXDC becomes viable as DeFi collateral** — the non-custodial design and ERC-4626 compliance make it suitable for lending markets and institutional integration
- **Reduced counterparty risk** — no single admin key can affect user balances
- **Transparent validator economics** — masternode operations are verifiable on-chain
- **Predictable governance** — all changes are time-locked and visible before they take effect
- **Standard interface** — ERC-4626 is the same vault interface used by major DeFi protocols, reducing integration effort

→ [V3 Architecture Details](v3-architecture.md)
→ [Institutional Overview](../../../institutional/README.md)
