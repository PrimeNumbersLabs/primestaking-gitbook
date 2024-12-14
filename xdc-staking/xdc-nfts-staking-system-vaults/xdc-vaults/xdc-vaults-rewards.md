# XDC Vaults Rewards

At Prime Numbers Labs, our XDC Vaults are designed to offer scalable, transparent, and rewarding staking opportunities for XDC holders. To ensure fair distribution of rewards while maintaining system stability, our APY adjusts dynamically based on the total amount of XDC staked and the number of active masternodes. The APY values listed are **minimum APY guarantees per staking tranche**, ensuring all participants benefit as staking levels grow.

***

#### **How APY Rewards Are Calculated**

The APY (Annual Percentage Yield) offered through XDC Vaults depends on two key factors:

1. **Total XDC Staked:** The total amount of XDC deposited into the Vaults by all participants.
2. **Number of Active Masternodes:** Each masternode can support up to 10M XDC, generating rewards at 8% APY (7.5% net after a 0.5% platform fee).

To ensure operational stability, the system maintains a **10M XDC buffer in the smart contract** at all times. This means the number of masternodes lags slightly behind the total staked XDC, ensuring efficient scaling and long-term sustainability.

***

#### **APY Reward Table (Minimum APY Per Tranche)**

| **Total XDC Staked**      | **Number of Masternodes** | **Buffer in Contract (XDC)** | **Minimum APY (%)** | **Explanation**                                             |
| ------------------------- | ------------------------- | ---------------------------- | ------------------- | ----------------------------------------------------------- |
| 0 - 10,000,000            | 1                         | 10,000,000                   | 7.50%               | 1 masternode fully covers rewards for the first 10M XDC.    |
| 10,000,001 - 20,000,000   | 1                         | 10,000,000                   | 3.75%               | 1 masternode split across 20M XDC, halving the APY.         |
| 20,000,001 - 30,000,000   | 2                         | 10,000,000                   | 5.00%               | 2 masternodes cover rewards, increasing APY.                |
| 30,000,001 - 40,000,000   | 3                         | 10,000,000                   | 5.62%               | 3 masternodes split across 40M XDC, improving APY further.  |
| 40,000,001 - 50,000,000   | 4                         | 10,000,000                   | 6.00%               | 4 masternodes split across 50M XDC, increasing APY.         |
| 50,000,001 - 60,000,000   | 5                         | 10,000,000                   | 6.25%               | 5 masternodes split across 60M XDC, slightly improving APY. |
| 60,000,001 - 70,000,000   | 6                         | 10,000,000                   | 6.43%               | 6 masternodes split across 70M XDC, improving APY further.  |
| 70,000,001 - 80,000,000   | 7                         | 10,000,000                   | 6.56%               | 7 masternodes split across 80M XDC, nearing full APY.       |
| 80,000,001 - 90,000,000   | 8                         | 10,000,000                   | 6.67%               | 8 masternodes split across 90M XDC, maximizing rewards.     |
| 90,000,001 - 100,000,000  | 9                         | 10,000,000                   | 6.75%               | 9 masternodes split across 100M XDC, nearing full capacity. |
| 100,000,001 - 110,000,000 | 10                        | 10,000,000                   | 6.82%               | 10 masternodes split across 110M XDC, stabilizing APY.      |

***

#### **How APY Adjusts with Staking Levels**

1. **Initial Stage (0-10M XDC):**
   * With one masternode fully covering up to 10M XDC, stakers receive the maximum APY of **7.5%**, the highest in this tranche.
2. **Scaling Beyond 10M XDC:**
   * Between **10M and 20M XDC**, the rewards are split across all stakers, ensuring a **minimum APY of 3.75%** for participants in this tranche.
   * Once the system adds a second masternode at **20M XDC**, the minimum APY rises to **5.0%**.
3. **Sustainable Growth Beyond 30M XDC:**
   * As the staking level exceeds **30M XDC**, additional masternodes are activated for every extra 10M XDC.
   * As staking levels grow, the minimum APY continues to rise with each new masternode, stabilizing near the maximum APY of 7.5%.

***

#### **Key Features of the XDC Vaults APY System**

1. **Minimum APY Guarantees:**
   * The listed APY values are the **minimum guaranteed for each tranche** based on the total XDC staked and the number of active masternodes.
   * Participants may earn higher APY as additional masternodes are activated and staking scales efficiently.
2. **10M XDC Buffer:**
   * The system retains a 10M XDC buffer in the contract to ensure operational stability and seamless masternode scaling.
3. **Scalable Rewards:**
   * New masternodes are added dynamically for every additional 10M XDC staked, improving APY over time.
4. **Transparency:**
   * All staking activity and APY calculations are on-chain, providing participants with full visibility and trust.

***

#### **Why Choose XDC Vaults for Staking?**

1. **Predictable Rewards:**
   * The system ensures participants earn **minimum APY guarantees per tranche**, with the potential for higher returns as staking grows.
2. **Liquidity with $pstXDC:**
   * Receive $pstXDC tokens for your staked XDC, enabling you to participate in DeFi applications without losing staking rewards.
3. **Security:**
   * Built on audited smart contracts, the Vaults provide a secure environment for staking.
4. **Sustainable Growth:**
   * The scalable system ensures that as more XDC is staked, rewards remain competitive and fair for all participants.

***
