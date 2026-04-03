# V2 vs V3 — What Changed and Why

{% hint style="warning" %}
V3 is currently completing its final security audit and will be deployed soon.
{% endhint %}

psXDC is migrating from its current custodial contract (V2) to a fully non-custodial, ERC-4626 vault architecture (V3). This page explains what changed, why, and what it means for users and partners.

---

## Why Migrate

V2 was the first generation of psXDC. It works, but its custodial design creates trust dependencies that limit adoption — particularly as collateral in DeFi lending markets. The core issues:

- **Withdrawals require admin approval** — a user requests to withdraw, then the owner must manually approve each request before XDC is released
- **XDC sits idle** — deposited XDC is not actively staked with validators
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
| **XDC utilization** | Idle — sits in the contract | Productive — staked with masternodes earning yield |
| **Masternode integration** | None | Direct on-chain integration with XDC validator contract |
| **Proxy pattern** | UUPS (owner-controlled upgrades) | TransparentUpgradeableProxy (admin/logic separation) |

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
| **How rewards enter** | Owner calls `notifyRewardAmount(reward)` with XDC | Validator rewards flow directly into the contract |
| **Distribution logic** | Time-based APY: `rewardRate * elapsed / totalStaked` | Asset-based: exchange rate = `totalAssets / totalShares` |
| **User experience** | Claim rewards manually from Rewards tab | No action — share value increases automatically |
| **Transparency** | Owner-controlled reward injection | Validator rewards verifiable on-chain via masternode contract |
| **Risk** | Owner sets APY and reward duration manually | Exchange rate driven by actual validator performance |

---

## Security Comparison

| Feature | V2 | V3 |
| --- | --- | --- |
| **Audit** | QuillAudits | QuillAudits + Nethermind Security |
| **Reentrancy protection** | Yes | Yes (OpenZeppelin ReentrancyGuard) |
| **Single point of failure** | Yes — single owner key | No — 5 roles + time-locks |
| **Governance delay** | None | 1-day minimum (configurable, max 30 days) |
| **Loss caps** | None | 10% per report, 20% per day |
| **Principal tracking** | None | Per-operator and global tracking |
| **Admin can drain funds** | Technically yes (`ownerWithdraw`, `mint`) | No — no such functions exist |
| **Zero slashing risk** | Yes (XDC model) | Yes (XDC model) |

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
| **Trust** | Trust the admin not to misuse `mint()` or `ownerWithdraw()` | Trust the code — no admin can move your funds |
| **DeFi use** | Hold, trade, or stake in NFTs | Same + usable as collateral in lending protocols |
| **Migration** | — | One-time: approve + migrate via bridge contract |

---

## What This Means for Partners

- **psXDC becomes viable as DeFi collateral** — the non-custodial design and ERC-4626 compliance make it suitable for lending markets and institutional integration
- **Reduced counterparty risk** — no single admin key can affect user balances
- **Transparent validator economics** — masternode operations are verifiable on-chain
- **Predictable governance** — all changes are time-locked and visible before they take effect
- **Standard interface** — ERC-4626 is the same vault interface used by major DeFi protocols, reducing integration effort

→ [V3 Architecture Details](v3-architecture.md)
→ [Institutional Overview](../../../institutional/README.md)
