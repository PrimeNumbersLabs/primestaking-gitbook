# Reward Mechanics

This section provides a technical overview of how XDC Liquid Staking rewards are generated, calculated, and distributed - the core economic engine that partners are integrating.

---

## Reward Source

Rewards come from **XDC Network validator staking**. PrimeStaking operates masternodes that participate in the XDC consensus mechanism and earn block rewards.

Unlike ETH-based liquid staking protocols, XDC Network **does not implement slashing**. This means staked capital is never at risk from validator behavior - a structural advantage for partners and their users.

---

## Why Use PrimeStaking vs. Direct Staking

Direct XDC staking requires running a masternode (10M XDC minimum, infrastructure management, uptime obligations). PrimeStaking removes all of these barriers:

| | Direct Staking | PrimeStaking |
| --- | --- | --- |
| **Minimum** | 10,000,000 XDC | None |
| **Infrastructure** | Run and maintain a masternode | Fully managed |
| **Liquidity** | Locked until unstake | Liquid (psXDC is transferable and tradeable) |
| **Slashing risk** | None (XDC model) | None (XDC model) |
| **Reward rate** | Depends on your node's uptime | Pooled across optimized validators |
| **Composability** | None | psXDC usable in DeFi |

For partners, this means their users can access XDC staking yield with zero infrastructure burden and no minimum deposit.

---

## Calculation

| Parameter | Detail |
| --- | --- |
| **Gross APY** | Determined by XDC Network validator economics |
| **Protocol fee** | Percentage retained by the protocol (contact us for specifics) |
| **Net user APY** | ~4.5% (after protocol fee) |
| **Distribution basis** | Proportional to psXDC balance |

The net APY is variable and depends on:

- **Network staking ratio** - total XDC staked across the network affects per-validator rewards
- **Validator performance** - uptime and block production efficiency
- **Protocol fee** - the percentage retained by the protocol before user distribution

---

## Distribution

- Rewards accrue continuously as validators produce blocks
- Users claim rewards manually from the Rewards contract
- Reward amounts are calculated proportionally based on psXDC holdings at claim time
- All distributions are on-chain and verifiable

---

## Key Parameters for Partners

| Parameter | Detail |
| --- | --- |
| **Reward token** | psXDC |
| **Frequency** | Continuous accrual |
| **APY** | ~4.5% (variable) |
| **Claiming** | Manual (user-initiated) |
| **On-chain verifiable** | Yes |
| **Variable/Fixed** | Variable (depends on validator performance and network conditions) |
| **Slashing risk** | None |

---

## How APY Is Determined

1. **Validator economics** - XDC Network masternodes earn block rewards based on the network's consensus parameters
2. **Protocol fee** - A percentage of gross validator rewards is retained by the protocol
3. **Net distribution** - Remaining rewards are distributed proportionally to all psXDC holders
4. **Dynamic rate** - APY fluctuates with network activity, validator count, and total staked XDC across the network

---

## Transparency & Verification

- Every reward event is logged on the XDC blockchain
- Reward calculations are deterministic and auditable
- Partners can independently verify reward distributions via on-chain data
- Historical reward data is available for forecasting and reporting
- psXDC supply and staking pool balance are verifiable on-chain at any time via [XDCScan](https://xdcscan.com/token/xdc59e51346a6e1d05a0017c0e5f0501dcbba41bca1)
