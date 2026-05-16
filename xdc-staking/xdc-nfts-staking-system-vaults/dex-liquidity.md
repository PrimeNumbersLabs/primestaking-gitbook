---
hidden: true
---

# DEX Liquidity

psXDC is paired with XDC on the **XSWAP DEX**, enabling trading and liquidity provision for participants in the XDC Liquid Staking ecosystem.

{% hint style="info" %}
The psXDC v3 share token (address [`0x98D9…C4Ba`](https://xdcscan.com/address/0x98D916F5773Ac0482b49856f2659d6c32114C4Ba)) is **NAV-based**, not a fixed 1:1 receipt. The market price on a DEX is set by AMM dynamics and may sit at, above, or below the vault's current exchange rate (`totalAssets / totalShares`). Long-term it tracks NAV; short-term it reflects supply, demand, and pool depth.
{% endhint %}

---

## Why Provide Liquidity?

| Reason | Detail |
| --- | --- |
| **Enable trading** | A liquid psXDC/XDC pool allows seamless swaps between the two tokens. |
| **Earn fees** | Liquidity providers earn trading fees on every swap in the pool. |
| **Support the ecosystem** | Deep liquidity strengthens psXDC utility and adoption, including as DeFi collateral. |
| **Earn appreciating exposure** | Half of your LP position is psXDC, whose NAV grows over time as the V3 vault accrues validator rewards. |

---

## How to Add Liquidity on XSWAP

### 1. Access XSWAP

Go to the [XSWAP platform](https://info.xspswap.finance/#/pools/0xc4a0b4ce176c623a281bc565bfd35eab4fd7050a) and connect your wallet.

### 2. Select the psXDC/XDC Pair

Find the liquidity pools section and search for the psXDC/XDC trading pair. Verify the pool's `psXDC` token address matches the V3 vault ([`0x98D9…C4Ba`](https://xdcscan.com/address/0x98D916F5773Ac0482b49856f2659d6c32114C4Ba)) before depositing — older pools may still hold the legacy V2 psXDC token (`0x9B8e12b0BAC165B86967E771d98B520Ec3F665A6`).

### 3. Prepare Tokens

Ensure you have an approximately equal value of psXDC v3 and XDC in your wallet. Note that the **value** of psXDC v3 ≠ 1 XDC — it equals the current vault exchange rate. The XSWAP UI accounts for this when computing how much of each token to deposit.

### 4. Add Liquidity

Enter the amounts you want to deposit and confirm the transaction. You will receive LP tokens representing your pool share.

### 5. Track Your Position

Monitor your position and earned fees through the XSWAP interface.

---

## Managing Your Position

- **Monitor NAV vs market price** — psXDC's protocol NAV grows over time. The DEX pool may not always track NAV in real-time; sustained gaps create arbitrage opportunities.
- **Impermanent loss** — be aware of IL risk if the NAV and pool ratio diverge significantly.
- **Rebalance** — add or withdraw liquidity periodically to maintain an optimal position.
- **Trading fees** — accrue automatically in your LP position. Withdraw LP tokens to claim.

---

## Withdrawing Liquidity

1. Go to the liquidity pool section on XSWAP.
2. Select your psXDC/XDC position.
3. Choose the amount to withdraw.
4. Confirm the transaction to receive your share of psXDC and XDC plus accrued fees.

---

## Protocol vs DEX exit — when to use which

| Path | When to use |
| --- | --- |
| **DEX swap (psXDC → XDC)** | You need XDC immediately and are willing to accept the current market price (which may include a small discount or premium to NAV). |
| **Protocol redeem (`redeemWithQueue`)** | You want the exact NAV value; settles instantly when the vault buffer covers it, otherwise via the on-chain FIFO queue. |

See [Withdrawals: Instant vs Queued](xdc-liquid-staking/staking-guide/withdrawals-instant-vs-queued.md) for how the protocol path chooses between instant and queued settlement.

---

## Best Practices

- Stay informed on staking rewards and market activity.
- Diversify across pools to mitigate risk.
- Compound returns by reinvesting trading fees.
- Follow [PrimeStaking](https://primestaking.xyz) and XSWAP updates for pool parameter changes.
