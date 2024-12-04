---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# PRFI NFT Staking Reward System

<mark style="color:red;">**Note:**</mark> <mark style="color:red;"></mark><mark style="color:red;">Our NFTs are being migrated to a new network.</mark>&#x20;

## <mark style="color:purple;">PRFI NFT STAKING REWARDS:</mark> <a href="#b642" id="b642"></a>

### 1- 100,000 PRFI Fixed Rewards

10M from the allocation is dedicated to rewarding PRFI NFT holders.&#x20;

Each month, 100,000 PRFI is distributed as a reward to these holders.

### 2- Prime Numbers NFTs Royalties <a href="#id-6649" id="id-6649"></a>

Using the NFT Royalty Standard, NFT holders will receive 50% of all royalties on presales.

The royalties percentage is set at 10%, which means 5% of the sales price of each sold NFT is distributed over all NFT holders.

### 3- PrimeFi <a href="#id-7896" id="id-7896"></a>

Prime Numbers NFTs will receive 40% of PrimeFi profits.&#x20;

{% hint style="info" %}
To be eligible for these rewards, you need to lock your NFT on PrimeFi for a period of time.
{% endhint %}

***

## <mark style="color:purple;">Calculating NFT Rewards with PRFI Staking Rewards Pool and Extra Rewards</mark>

### <mark style="color:blue;">Variables Overview</mark>

* **X**: The total number of NFTs in consideration.
* **a**: Rarity of the NFT.
* **b**: Level of the NFT.
* **t**: PRFI tokens assigned per NFT.
* **Tx**: Total multiplier for an NFT, calculated as (Tx = a + b).
* **Extra Rewards**: Additional rewards coming from Primeport and PrimeFi fees, added to the initial 100,000 PRFI rewards pool.

A total of 100,000 PRFI tokens, plus Extra Rewards, are to be distributed, taking into account both the NFTs' total multiplier and the product of the total multiplier and PRFI tokens per NFT.

***

### <mark style="color:blue;">Reward Distribution Formula</mark>

#### Part 1: Based on Total Multiplier

To distribute the first portion of PRFI tokens plus a share of the Extra Rewards, we calculate each NFT's share based on its total multiplier (Tx).

1. **Calculate the accumulated total multiplier**:

$$
T_{total} = \sum_{i=1}^{X} Tx_i
$$

2. **Determine the reward for each NFT**, also considering a portion of the Extra Rewards:

$$
R_{1,i} = \left(\frac{Tx_i}{T_{total}} \times 50000\right) + \left(\frac{Tx_i}{T_{total}} \times \text{Extra Rewards}_{\text{portion}}\right)
$$

#### Part 2: Based on Total Multiplier and PRFI Tokens

For the second distribution of PRFI tokens plus another share of the Extra Rewards, we include the product of the total multiplier and PRFI tokens (Tx \* t).

1. **Calculate the total of the product of total multipliers and PRFI tokens (Ptotal)**:

$$
P_{total} = \sum_{i=1}^{X} (Tx_i \times t_i)
$$

2. **Determine the reward for each NFT**, incorporating a portion of the Extra Rewards:

$$
R_{2,i} = \left(\frac{Tx_i \times t_i}{P_{total}} \times 50000\right) + \left(\frac{Tx_i \times t_i}{P_{total}} \times \text{Extra Rewards}_{\text{portion}}\right)
$$

***

### <mark style="color:blue;">Conclusion</mark>

By applying these formulas, each NFT's reward is determined by its unique characteristics (rarity and level) and the assigned PRFI tokens. This dual approach ensures a balanced reward distribution that considers both the intrinsic qualities of the NFTs and the PRFI tokens allocated to them.



