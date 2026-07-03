# DEX Liquidity & Spot Trading

psXDC can be traded against XDC on-chain, and liquidity providers can earn fees by supplying both sides of the pool.

The primary venue is **PrimeStaking Spot** at [primestaking.xyz/spot](https://primestaking.xyz/spot). It runs on a protocol-owned Uniswap V2 style pool holding the live V3.1 psXDC and wrapped XDC, with market swaps and on-chain limit orders settled by a keeper.

{% hint style="warning" %}
**Post-V3.1 note (July 2026):** DEX pools created before the V3.1 redeployment hold the retired old-V3 token and are no longer valid. LP positions in those pools were credited in the snapshot airdrop, so LPs received their share of the pool's psXDC directly on V3.1. Before trading or LPing anywhere, verify the pool's psXDC address is the live V3.1 token: [`0xa7FD1c5601348633018003C90aE568d1ff7973e4`](https://xdcscan.com/address/0xa7FD1c5601348633018003C90aE568d1ff7973e4).
{% endhint %}

{% hint style="info" %}
The psXDC share token is **NAV-based**, not a fixed 1:1 receipt. The market price on a DEX is set by AMM dynamics and may sit at, above, or below the vault's current exchange rate (`totalAssets / totalShares`). Long-term it tracks NAV; short-term it reflects supply, demand, and pool depth.
{% endhint %}

---

## The Spot venue at a glance

| Contract | Address |
| --- | --- |
| psXDC/WXDC pair (V3.1) | [`0x14A11af8980ea7e18B18cAbf4721B61586Bab087`](https://xdcscan.com/address/0x14A11af8980ea7e18B18cAbf4721B61586Bab087) |
| SpotLimitOrders (V3.1) | [`0x89dB7715fFc5B8b2C4A604BdD49b006df201247a`](https://xdcscan.com/address/0x89dB7715fFc5B8b2C4A604BdD49b006df201247a) |
| Router (UniswapV2Router02) | [`0xf77440C4Dc3Dcd5Bb93DaA863BF93Fc306EC0791`](https://xdcscan.com/address/0xf77440C4Dc3Dcd5Bb93DaA863BF93Fc306EC0791) |
| Factory (UniswapV2Factory) | [`0x3a718EB1b4b06968F78a0a3b7e3dF07037E83f5d`](https://xdcscan.com/address/0x3a718EB1b4b06968F78a0a3b7e3dF07037E83f5d) |

The pool was seeded at NAV (1 psXDC : 1 XDC at launch of V3.1). Limit orders are non-custodial: makers escrow only their unfilled input, the contract enforces the maker's price on every fill, and orders can always be cancelled for a full refund of the remaining escrow.

---

## Why Provide Liquidity?

| Reason | Detail |
| --- | --- |
| **Enable trading** | A liquid psXDC/XDC pool allows seamless swaps between the two tokens. |
| **Earn fees** | Liquidity providers earn trading fees on every swap in the pool. |
| **Support the ecosystem** | Deep liquidity strengthens psXDC utility and adoption, including as DeFi collateral. |
| **Earn appreciating exposure** | Half of your LP position is psXDC, whose NAV grows over time as the V3.1 vault accrues validator rewards. |

---

## How to Trade on PrimeStaking Spot

1. Go to [primestaking.xyz/spot](https://primestaking.xyz/spot) and connect your wallet.
2. **Market orders** swap immediately against the pool at the current price.
3. **Limit orders** rest on-chain until the pool price reaches your limit; a keeper then settles them automatically. You can cancel an open order at any time and the unfilled escrow is refunded in full.

## How to Add Liquidity

Liquidity for the psXDC/WXDC pair is added through the router contract. If you use a third-party DEX UI instead, verify the pool's `psXDC` token address matches the live V3.1 vault ([`0xa7FD…73e4`](https://xdcscan.com/address/0xa7FD1c5601348633018003C90aE568d1ff7973e4)) before depositing. Older pools hold the retired old-V3 token (`0x98D9…C4Ba`) or the legacy V2 token (`0x9B8e…65A6`).

When adding liquidity, remember that the **value** of 1 psXDC is not 1 XDC; it equals the current vault exchange rate, and pool ratios follow the market price.

---

## Managing Your Position

- **Monitor NAV vs market price.** psXDC's protocol NAV grows over time. The DEX pool may not always track NAV in real-time; sustained gaps create arbitrage opportunities.
- **Impermanent loss.** Be aware of IL risk if the NAV and pool ratio diverge significantly.
- **Rebalance.** Add or withdraw liquidity periodically to maintain an optimal position.
- **Trading fees** accrue automatically in your LP position. Withdraw LP tokens to claim.

---

## Withdrawing Liquidity

Burn your LP tokens through the router (or the DEX UI you used to deposit) to receive your share of psXDC and WXDC plus accrued fees.

{% hint style="info" %}
**Held LP tokens in the old (pre-V3.1) pool?** Your psXDC side was already credited in the snapshot airdrop. The WXDC side is still yours: remove liquidity from the old pair to recover it. The old psXDC you receive alongside it is the retired token and can be ignored.
{% endhint %}

---

## Protocol vs DEX exit: when to use which

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
- Follow [PrimeStaking](https://primestaking.xyz) for pool parameter changes.
