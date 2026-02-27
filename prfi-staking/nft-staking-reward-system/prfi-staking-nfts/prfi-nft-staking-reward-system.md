# Reward System

PRFI NFT holders earn from three distinct reward sources.

---

## Reward Sources

### 1. PRFI Fixed Rewards (100,000 PRFI/month)

10M PRFI from the token allocation is dedicated to rewarding NFT holders. Each month, 100,000 PRFI is distributed across all staked NFTs.

### 2. NFT Royalties

NFT holders receive 50% of all PrimePort marketplace royalties on secondary sales. The royalty rate is 10%, meaning 5% of each sale price is distributed to all NFT holders.

### 3. PrimeFi Profits

PRFI NFTs receive 40% of PrimeFi lending and borrowing protocol profits.

{% hint style="info" %}
To be eligible for PrimeFi rewards, you need to lock your NFT on PrimeFi for a specified period.
{% endhint %}

---

## Reward Distribution Formula

Rewards are split into two equal portions:

### Part 1 - Based on Total Multiplier (50%)

Each NFT's share is proportional to its total multiplier relative to the sum of all multipliers:

$$
R_{1,i} = \frac{Tx_i}{T_{total}} \times 50{,}000 \text{ PRFI}
$$

Where:
- `Tx_i` = Total multiplier of NFT *i* (rarity + level)
- `T_total` = Sum of all NFTs' total multipliers

### Part 2 - Based on Multiplier x Stake (50%)

Each NFT's share also accounts for how much PRFI is staked:

$$
R_{2,i} = \frac{Tx_i \times t_i}{P_{total}} \times 50{,}000 \text{ PRFI}
$$

Where:
- `t_i` = PRFI tokens staked in NFT *i*
- `P_total` = Sum of (multiplier x stake) across all NFTs

### Total Reward

$$
R_i = R_{1,i} + R_{2,i} + \text{Extra Rewards share}
$$

Extra Rewards (royalties + PrimeFi profits) are distributed using the same proportional formulas.

---

## Summary

This dual approach ensures balanced reward distribution:

- **Part 1** rewards NFT quality (rarity and level).
- **Part 2** rewards staking commitment (multiplier x stake).

Both factors matter - a high-rarity NFT with a large stake earns the most.
