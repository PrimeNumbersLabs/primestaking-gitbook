# Reward Mechanics

This section provides a technical overview of how staking rewards are generated, calculated, and distributed across PrimeStaking's products.

---

## XDC Liquid Staking Rewards

### Source

Rewards come from **XDC Network validator staking**. PrimeStaking operates masternodes that participate in the XDC consensus mechanism and earn block rewards.

### Calculation

| Parameter | Detail |
| --- | --- |
| **Gross APY** | Determined by XDC Network validator economics |
| **Protocol fee** | Percentage retained by the protocol |
| **Net user APY** | ~4.5% (after protocol fee) |
| **Distribution basis** | Proportional to psXDC balance |

### Distribution

- Rewards accrue continuously as validators produce blocks
- Users claim rewards manually from the Rewards contract
- Reward amounts are calculated proportionally based on psXDC holdings at claim time
- All distributions are on-chain and verifiable

---

## XDC NFT Rewards

### Base APY

psXDC deposited inside XDC NFTs earns the standard liquid staking yield (~4.5%) plus an NFT staking bonus.

### APY Tiers

| Tier | APY | Requirement |
| --- | --- | --- |
| **Unlocked NFT** | ~4.75% | Deposit psXDC into an XDC NFT |
| **Locked NFT (1 year)** | Up to 6% | Lock the NFT for 1 year |

### Multiplier System

Each XDC NFT has a **Total Multiplier** = Base Multiplier (from rarity) + Added Multiplier (from level).

| Rarity | Base Multiplier |
| --- | --- |
| Plentiful | 0.3 |
| Common | 0.4 |
| Uncommon | 0.5 |
| Rare | 0.7 |
| Epic | 0.9 |
| Legendary | 1.2 |
| Mythic | 1.5 |
| Godly | 1.9 |

The NFT's share of the monthly reward pool is proportional to its Total Multiplier relative to the sum of all NFT multipliers.

### Distribution

- Rewards are paid in **XDC**
- Distribution is **monthly**
- Claims are manual via the app

---

## PRFI NFT Rewards

### Sources

| Source | Amount/Rate |
| --- | --- |
| **PRFI Rewards Pool** | 100,000 PRFI/month |
| **NFT Royalties** | 50% of PrimePort marketplace royalties (5% of sales) |
| **PrimeFi Profits** | 40% of PrimeFi protocol profits |

### Distribution Formula

Rewards are split into two equal portions:

**Part 1 — Based on Total Multiplier (50%)**

Each NFT receives a share proportional to its Total Multiplier relative to the sum of all multipliers.

**Part 2 — Based on Multiplier × Stake (50%)**

Each NFT receives a share proportional to (Total Multiplier × Staked PRFI) relative to the sum across all NFTs.

This dual approach rewards both NFT quality (rarity/level) and staking commitment (amount staked).

### Distribution

- Rewards are paid in **PRFI**
- Distribution is **monthly**
- Claims are available on any supported chain (omnichain via LayerZero)

---

## Key Parameters for Partners

| Parameter | XDC Liquid Staking | XDC NFTs | PRFI NFTs |
| --- | --- | --- | --- |
| **Reward token** | psXDC | XDC | PRFI |
| **Frequency** | Continuous accrual | Monthly | Monthly |
| **APY range** | ~4.5% | 4.75%–6% | Variable (pool-based) |
| **Claiming** | Manual | Manual | Manual |
| **On-chain verifiable** | Yes | Yes | Yes |
| **Variable/Fixed** | Variable (validator-dependent) | Variable (multiplier-dependent) | Fixed pool + variable extras |
