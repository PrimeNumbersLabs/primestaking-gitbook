# XDC Liquid Staking System

The XDC Liquid Staking System is PrimeStaking's core infrastructure. It enables XDC holders to earn staking rewards without running infrastructure or locking funds permanently.

The V3 stack is fully non-custodial: psXDC is an ERC-4626 vault share, withdrawals are self-service, and the staking vault contract cannot be modified by anyone after deployment.

---

## How It Works

```
Stake XDC  →  Receive psXDC shares  →  Share price grows (~4.5% APY)  →  Withdraw anytime
```

1. **Deposit XDC** into the [`PrimeStakedXDC_V3`](contract-addresses.md) vault.
2. **Receive psXDC shares** at the current vault exchange rate. There is no fixed 1:1; share value rises as validator rewards accrue.
3. **Earn rewards automatically** — rewards are embedded in the share price. No claim button for the base yield.
4. **Use psXDC** — it's a standard ERC-20 on the XDC Network. Hold it, trade it, use it as collateral, or stake it inside an XDC NFT for an additional boost slice.
5. **Withdraw** — burn psXDC shares. If the vault has enough buffer liquidity you receive XDC instantly; otherwise your request enters an automatic FIFO queue and you claim when masternode payouts return.

---

## Two Ways to Earn

### XDC Liquid Staking (~4.5% APY)

The simplest path. Stake XDC, receive psXDC shares, watch the share price climb. No NFT required, no minimum amount.

→ [XDC Liquid Staking Details](xdc-liquid-staking/)

### XDC NFTs (Base NAV + Boost)

Gamified staking on top of liquid staking. Deposit psXDC shares into collectible NFTs and earn **two stacked yields**:

- The underlying psXDC NAV (same ~4.5% the vault already gives you).
- A Synthetix-style **boost slice** funded by the protocol's [`XdcNftBoostHarvester`](xdc-staking-nfts/boost-harvester.md), weighted by NFT rarity, level, and lock status.

**Floor: ~4.5%** — the base NAV is always there, automatic, no claim. When the boost stream is flowing, the combined APY ranges from **~4.75% unlocked** up to **~6% locked**.

→ [XDC NFTs Details](xdc-staking-nfts/)

---

## What Is psXDC?

**psXDC (Prime Staked XDC)** is the ERC-4626 vault share you receive when you stake XDC in V3.

| Property | Detail |
| --- | --- |
| Standard | ERC-4626 tokenized vault share |
| Pricing | Exchange rate `totalAssets / totalShares` — grows over time as rewards accrue (not a fixed 1:1) |
| Network | XDC Network (chain ID `50`) |
| Transferable | Yes — send, trade, or use as DeFi collateral |
| Earns rewards | Yes — holding psXDC means the share is worth more XDC each block |
| Redeemable | Yes — burn psXDC to withdraw XDC, instantly if the buffer allows, otherwise via the automatic FIFO queue |
| Address | [`0x98D916F5773Ac0482b49856f2659d6c32114C4Ba`](https://xdcscan.com/address/0x98D916F5773Ac0482b49856f2659d6c32114C4Ba) |

psXDC is the entry point to the entire staking ecosystem. You can hold it as-is for the base ~4.5% APY, or deposit it into XDC NFTs to layer the boost slice on top.

→ [V3 Architecture](xdc-liquid-staking/v3-architecture.md) → [Deployed Contracts](contract-addresses.md)
