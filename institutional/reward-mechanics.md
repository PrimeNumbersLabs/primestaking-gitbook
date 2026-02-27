# Reward Mechanics

This section provides a technical overview of how XDC Liquid Staking rewards are generated, calculated, and distributed.

---

## Reward Source

Rewards come from **XDC Network validator staking**. PrimeStaking operates masternodes that participate in the XDC consensus mechanism and earn block rewards.

---

## Calculation

| Parameter | Detail |
| --- | --- |
| **Gross APY** | Determined by XDC Network validator economics |
| **Protocol fee** | Percentage retained by the protocol |
| **Net user APY** | ~4.5% (after protocol fee) |
| **Distribution basis** | Proportional to psXDC balance |

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
| **APY** | ~4.5% |
| **Claiming** | Manual (user-initiated) |
| **On-chain verifiable** | Yes |
| **Variable/Fixed** | Variable (depends on validator performance and network conditions) |

---

## How APY Is Determined

1. **Validator economics** - XDC Network masternodes earn block rewards based on the network's consensus parameters
2. **Protocol fee** - A percentage of gross validator rewards is retained by the protocol
3. **Net distribution** - Remaining rewards are distributed proportionally to all psXDC holders
4. **Dynamic rate** - APY fluctuates with network activity, validator count, and total staked XDC across the network

---

## Transparency

- Every reward event is logged on the XDC blockchain
- Reward calculations are deterministic and auditable
- Partners can independently verify reward distributions via on-chain data
- Historical reward data is available for forecasting and reporting
