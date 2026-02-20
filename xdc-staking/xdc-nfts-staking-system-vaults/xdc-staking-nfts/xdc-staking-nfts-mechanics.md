# Staking Mechanics

XDC Staking NFTs use a multiplier-based system. The more you stake and the higher your NFT's rarity, the greater your share of the monthly reward pool.

---

## How Multipliers Work

Each NFT has two multiplier components:

| Component | Description |
| --- | --- |
| **Base Multiplier** | Determined by the NFT's rarity tier (fixed per rarity). |
| **Added Multiplier** | Increases as you level up the NFT by staking more psXDC. |
| **Total Multiplier** | Base + Added. Determines your reward share. |

{% hint style="info" %}
Total Multiplier = Base Multiplier + Added Multiplier
{% endhint %}

**Example:** A Rare NFT (base 0.7) at Level 18 (added 1.8) has a Total Multiplier of **2.5**.

---

## NFT Structure

**Base Multiplier** — Determined by rarity:

<figure><img src="../../../.gitbook/assets/BaseMultiplierXDC (4).jpg" alt=""><figcaption></figcaption></figure>

**Added Multiplier** — Increases as you level up:

<figure><img src="../../../.gitbook/assets/AddedMultiplierXDC (2).jpg" alt=""><figcaption></figcaption></figure>

---

## Staking psXDC Inside an NFT

- **What you stake:** psXDC (acquired by staking XDC or buying on a DEX).
- **Max per NFT:** 100,000 psXDC.
- **Leveling:** Staking more psXDC increases the NFT's level and added multiplier.
- **Withdrawal:** psXDC can be withdrawn at any time (unless the NFT is locked).

---

## Locking for Extra Yield

Lock your NFT for **1 year** for an additional **1.25% APY** bonus, bringing the maximum to **6% APY**.

**When locked:**
- Burn, merge, and withdraw functions are disabled.
- The NFT continues earning rewards at the boosted rate.
- Rewards are claimable monthly.

---

## Interface Actions

| Action | Description |
| --- | --- |
| **Stake** | Deposit psXDC into your NFT to earn rewards. |
| **Withdraw psXDC** | Remove staked psXDC without destroying the NFT. |
| **Transfer** | Move the NFT to another wallet. |
| **Merge** | Combine two same-rarity NFTs into a higher rarity. |
| **Lock** | Lock the NFT for 1 year for +1.25% APY (disables burn/merge/withdraw). |
| **Claim XDC** | Claim your monthly XDC rewards. |
| **Sell** | List or auction the NFT on [PrimePort](https://primeport.xyz). |
