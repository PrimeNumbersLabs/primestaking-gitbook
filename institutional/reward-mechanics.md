# Reward Mechanics

This section explains how XDC liquid staking rewards are generated and distributed under V3. The model is **NAV-based**: rewards are not handed out via a manual `claim` flow — they are embedded in the psXDC vault share price.

---

## Reward Source

PrimeStaking generates yield through XDC Network masternode operations. Validators participate in the XDPoS consensus process and receive protocol rewards derived from block production and network activity.

Unlike ETH-based liquid staking protocols, XDC Network **does not implement punitive principal-slashing mechanisms comparable to Ethereum Casper**. Validator penalties are limited to operational demotion and reward impacts, without destruction of the underlying staked capital. This creates a materially lower staking risk profile for institutional partners and end users.

---

## Why Use PrimeStaking vs. Direct Staking

Direct XDC staking requires running a masternode (10M XDC minimum, infrastructure management, uptime obligations). PrimeStaking removes all of these barriers:

| | Direct Staking | PrimeStaking V3 |
| --- | --- | --- |
| **Minimum** | 10,000,000 XDC | None |
| **Infrastructure** | Run and maintain a masternode | Fully managed |
| **Liquidity** | Locked until unstake | Liquid (psXDC is ERC-4626, transferable, tradeable) |
| **Slashing risk** | None (XDC model) | None (XDC model) |
| **Reward rate** | Depends on your node's uptime | Pooled across optimized validators |
| **Composability** | None | psXDC usable as ERC-4626 collateral in DeFi |
| **Withdrawal UX** | Wait the network unstake delay | Instant when buffer allows; FIFO queue otherwise |

---

## How rewards reach holders

```
XDC validators
      │ block rewards
      ▼
PrimeStakedXDC_V3 vault
      │ totalAssets += rewards
      │ totalShares  unchanged
      ▼
exchange rate (totalAssets / totalShares) ↑
      │
      ▼
every psXDC share is worth more XDC
```

| Aspect | Detail |
| --- | --- |
| **Source** | XDC Network masternode block rewards |
| **Accrual mechanism** | Reward XDC enters the vault → `totalAssets` rises → exchange rate rises automatically |
| **User claiming** | None — value is already inside each share |
| **Settlement event** | When the user redeems shares (instant or queued), the higher rate translates directly into more XDC returned |
| **On-chain verifiability** | Yes — every reward inflow event and the exchange rate are public |

There is **no `notifyRewardAmount` admin call** in V3 (V2-era behaviour). There is **no per-user `claim` flow** for the base reward layer. Both were removed when V3 replaced the time-based APY model with the share-based NAV model.

---

## Calculation

| Parameter | Detail |
| --- | --- |
| **Gross APY** | Determined by XDC Network masternode economics (network staking ratio, validator performance, operator throughput) |
| **Protocol fee** | Percentage of gross validator rewards retained by the protocol — exact figure available under partner due diligence |
| **Net user APY** | ~4.5% net (variable; depends on the above) |
| **Distribution basis** | Pro-rata over psXDC shares **automatically through share price**, not at claim time |

Net APY is variable and depends on:

- **Network staking ratio** — total XDC staked across the network affects per-validator rewards.
- **Validator performance** — uptime and block production efficiency for the operators the vault is delegating to.
- **Protocol fee** — retained percentage before reward XDC is reflected in `totalAssets`.

---

## NFT boost layer (separate from base APY)

XDC NFTs earn an **additional** XDC stream on top of base NAV via the Synthetix-style accumulator inside `XdcNftStakingVault`. The boost is **separate** from validator rewards and follows its own funding model:

- Boost is pushed into the NFT vault by [`XdcNftBoostHarvester`](../xdc-staking/xdc-nfts-staking-system-vaults/xdc-staking-nfts/boost-harvester.md) via `notifyBoost(amount)`.
- The boost slice is distributed pro-rata to each NFT's weight (`stakedShares × (rarityMultiplier + level + lockBonus)`).
- Boost **is** claimed (`claim(tokenId)`) and paid in XDC.

Boost is a product-side reward stream, not validator economics. Target band for NFT positions is **~4.75% (Plentiful unlocked) → ~6% (Handcrafted locked)** combining base NAV + boost.

→ [Reward Model: Base NAV + Boost](../xdc-staking/xdc-nfts-staking-system-vaults/xdc-staking-nfts/xdc-nft-staking-reward-system.md)

---

## Key Parameters for Partners

| Parameter | Detail |
| --- | --- |
| **Reward asset (base layer)** | XDC, accruing as share-price appreciation of psXDC |
| **Reward asset (NFT boost)** | XDC, accruing into the NFT vault's Synthetix accumulator |
| **Distribution frequency (base)** | Continuous via share-price growth (no batches) |
| **Distribution frequency (boost)** | Each `notifyBoost` event — cadence is an operational choice (typically weekly or daily) |
| **Claim flow (base)** | None — rewards are realized on redemption |
| **Claim flow (boost)** | User-initiated `claim(tokenId)` from the NFT detail page |
| **On-chain verifiability** | Yes — both layers emit events indexed by the public subgraphs |
| **Slashing risk** | None |

---

## Loss Reporting

Validator outcomes can be reported via `reportValidatorLoss(operator, assets)` (gated by `RISK_MANAGER_ROLE`). The function is bounded by:

- `maxLossBpsPerReport` — cap per individual report.
- `maxDailyLossBps` — cap over a rolling 24h window.

Both caps are themselves governed by **delayed governance** — changes are scheduled, wait for `governanceDelay`, then execute. Reports emit the attributed operator (`outstandingValidatorPrincipalByOperator` is updated atomically). This bounds the blast radius of any single risk-management call.

---

## Transparency & Verification

- Every reward event is logged on the XDC blockchain.
- Exchange rate is a deterministic function of `totalAssets` and `totalShares` — auditable at any block.
- Historical exchange-rate data is available via the public subgraph for forecasting and reporting.
- psXDC v3 supply and total assets are verifiable on-chain at any time on [XDCScan](https://xdcscan.com/address/0x98D916F5773Ac0482b49856f2659d6c32114C4Ba).

→ [Liquidity Model](liquidity-model.md) → [Risk & Compliance](risk-and-compliance.md) → [How Rewards Work (user-facing)](../xdc-staking/xdc-nfts-staking-system-vaults/xdc-liquid-staking/xdc-staking-rewards.md)
