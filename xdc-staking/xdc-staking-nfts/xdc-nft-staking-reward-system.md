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

# XDC NFT Staking Reward System

## <mark style="color:purple;">XDC NFTs STAKING REWARDS:</mark> <a href="#b642" id="b642"></a>

### 1- Fixed Rewards

In the first two years, 30% of the funds from the minting will be rewarded to the XDC Staking NFTs.

### 2.- XDC NFTs Masternodes Staking - Optional

When your NFT reaches the MAX LIMIT (100,000 $XDC staked inside the NFT), you can lock this NFT to get an additional 7% APY.&#x20;

The lock period will be one year. The reward will be paid monthly to the NFT.

{% hint style="success" %}
Example: You have an NFT with 100 000 $XDC. And you lock it.\
From the lock, you get 7% of 100 000 $XDC -> 7000 $XDC.
{% endhint %}

***

## <mark style="color:purple;">Calculating NFT Rewards with PRNT Staking Rewards Pool and Extra Rewards</mark>

### <mark style="color:purple;">Variables Overview</mark>

* **X**: The total number of NFTs in consideration.
* **a**: Rarity of the NFT.
* **b**: Level of the NFT.
* **t**: XDC tokens assigned per NFT.
* **Tx**: Total multiplier for an NFT, calculated as (Tx = a + b).
* **Extra Rewards**: Additional rewards coming from Prime Finance fees, added to the initial minting funds rewards.

A total of 82,000 tokens, plus Extra Rewards, are to be distributed, taking into account both the NFTs' total multiplier and the product of the total multiplier and XDC tokens per NFT.

***

### <mark style="color:purple;">Reward Distribution Formula</mark>

#### Part 1: Based on Total Multiplier

To distribute the first portion of XDC tokens plus a share of the Extra Rewards, we calculate each NFT's share based on its total multiplier (Tx).

1. **Calculate the accumulated total multiplier**:

$$
T_{total} = \sum_{i=1}^{X} Tx_i
$$

2. **Determine the reward for each NFT**, also considering a portion of the Extra Rewards:

$$
R_{1,i} = \left(\frac{Tx_i}{T_{total}} \times 82000\right) + \left(\frac{Tx_i}{T_{total}} \times \text{Extra Rewards}_{\text{portion}}\right)
$$



#### Part 2: Based on Total Multiplier and XDC Tokens

For the second distribution of XDC tokens plus another share of the Extra Rewards, we include the product of the total multiplier and XDC tokens (Tx \* t).

1. **Calculate the total of the product of total multipliers and XDC tokens (Ptotal)**:

$$
P_{total} = \sum_{i=1}^{X} (Tx_i \times t_i)
$$

2. **Determine the reward for each NFT**, incorporating a portion of the Extra Rewards:

$$
R_{2,i} = \left(\frac{Tx_i \times t_i}{P_{total}} \times 82000\right) + \left(\frac{Tx_i \times t_i}{P_{total}} \times \text{Extra Rewards}_{\text{portion}}\right)
$$

***

### <mark style="color:purple;">Conclusion</mark>

Applying these formulas determines each NFT's reward by its unique characteristics (rarity and level) and the assigned XDC tokens.&#x20;

