# FAQs

### General Questions

#### What is XDC Liquid Staking?

**XDC Liquid Staking** enables you to stake XDC **while retaining full liquidity**. Instead of locking up your tokens, you receive psXDC (Prime Staked XDC), an ERC-4626 vault share that can be freely used or traded in DeFi.

* **XDC Liquid Staking:** A straightforward way to stake XDC and receive psXDC vault shares, with no minimum required. Earns ~4.5% APY through share-price appreciation.
* **XDC NFTs:** Deposit psXDC shares inside XDC NFTs to layer a boost slice on top of the base NAV. **Floor stays at the base ~4.5%** (always earned, regardless of rarity / lock / boost cadence); when the boost stream is flowing, the combined APY ranges from **~4.75% (unlocked)** up to **~6% (locked)**.

#### What is psXDC?

psXDC is an **ERC-4626 vault share** representing your position in the V3 staking vault. You don't claim rewards separately. The share grows in value over time (`totalAssets / totalShares` increases as validator rewards accrue), so each psXDC is worth more XDC the longer you hold it.

#### Where can I trade psXDC?

psXDC can be traded on DEXs on the XDC Network, paired with XDC. **After the July 2026 V3.1 redeployment, verify any pool holds the live token** ([`0xa7FD…73e4`](https://xdcscan.com/address/0xa7FD1c5601348633018003C90aE568d1ff7973e4)); pools created before the redeployment hold retired tokens. Purchasing psXDC directly also benefits you, since holding it means owning the appreciating share.

#### I held psXDC V3 before July 2026. Do I need to do anything?

No. The vault was redeployed as **V3.1** and every V3 balance was mirrored 1:1 via an on-chain snapshot airdrop, including psXDC inside XDC NFTs, DEX pools, lending markets, and open limit orders, which were credited to their underlying owners. Your balance appears automatically in the app. The old V3 token is retired and has no remaining function.

#### Why were migrations briefly unavailable during the V3.1 cutover (July 3-4, 2026)?

By design. During the cutover the V2 token was paused and the final activation steps (retiring the old V3 bridge and pointing the V3.1 vault at the new migration bridge) sat behind a 24-hour on-chain timelock, a deliberate safety window that made it impossible for anyone to migrate into the retired old V3 token by mistake. Once the timelock matured on July 4 the bridge was activated, V2 was unpaused, and both token and NFT migrations reopened. No funds were ever at risk and nothing expired during the window.

#### What happens when I unstake?

When you decide to unstake, you burn the corresponding psXDC shares. The app calls `redeemWithQueue` which:

- **Settles instantly** when the vault's liquid buffer (default 5% of total assets) covers your request. XDC returns to your wallet in the same transaction.
- **Enters a permissionless FIFO queue** otherwise: your shares are escrowed and a request is created. As soon as the buffer is replenished (new deposits, reward inflows, masternode resignations) the queue settles your request. For very large redemptions the upper bound is the network's `candidateWithdrawDelay`, approximately **~35 days** under typical block times.

You can cancel queued requests any time, and if a payout ever fails the XDC lands in `pendingQueuedAssets` so you can collect it via `claimQueuedAssets`.

→ [Withdrawals: Instant vs Queued](xdc-liquid-staking/staking-guide/withdrawals-instant-vs-queued.md)

#### Can I transfer my staking position?

Yes.

* **Liquid Staking:** Send psXDC to another address; the recipient inherits the appreciating share. No further action needed.
* **XDC NFTs:** If your psXDC is staked inside an XDC NFT, you can sell or transfer the NFT via [PrimePort.xyz](https://primeport.xyz). The new owner inherits the staked shares, pending boost, and any lock status.

#### How are rewards distributed?

| Product | Base reward | How you receive it |
| --- | --- | --- |
| **XDC Liquid Staking** | ~4.5% via psXDC share-price growth | **Automatic**, with no claim button. Rewards are realized when you redeem or transfer the share. |
| **XDC NFTs** | Base ~4.5% (NAV) + boost slice (up to ~1.5% via Synthetix accumulator) | **Base is automatic** (same as liquid). **Boost is claimed** from the NFT detail page in the app, paid in XDC. |

→ [How Rewards Work](xdc-liquid-staking/xdc-staking-rewards.md) → [Reward Model: Base NAV + Boost](xdc-staking-nfts/xdc-nft-staking-reward-system.md)

#### Is there any risk of slashing?

XDC Network does have a slashing mechanism, but it differs fundamentally from Ethereum's. A masternode that fails to sign any block during one full epoch (900 blocks, ~30 minutes) is excluded from block production for the next 4 epochs (~2 hours) and forfeits rewards during that window. **Crucially, principal stake is not burned**: staked capital is never destroyed by validator behavior. For psXDC holders this means a slashing event on a protocol masternode would only briefly slow the rate at which the share price grows during the exclusion window; it cannot reduce the value of the shares themselves. This is structurally different from Ethereum-based liquid staking, where slashing can permanently destroy a portion of staked ETH.

***

### XDC NFTs

#### What are XDC Staking NFTs?

**XDC Staking NFTs** offer a gamified approach to staking. You deposit psXDC shares into an NFT, and the NFT's rarity, level, and lock status determine its weight in the boost accumulator. Higher weight = larger slice of every boost push.

#### How do XDC NFTs work?

1. **Deposit psXDC shares:** acquire psXDC (by staking XDC or buying on a DEX) and deposit it into your NFT.
2. **Earn base NAV:** the underlying psXDC shares keep appreciating as the vault accrues validator rewards, the same ~4.5% you'd get without the NFT.
3. **Earn boost:** each `notifyBoost` push from the protocol's harvester credits the Synthetix accumulator; your NFT's pending boost grows proportionally to its weight.
4. **Claim boost:** from the NFT detail page, in XDC.
5. **Merge:** combine two NFTs of the same rarity to create a higher-rarity NFT with a bigger weight.
6. **Lock (optional):** locking adds `lockBonus` to the weight, which can push the combined APY toward the ~6% top of the band, but disables withdraw / merge / `burnAndRedeem` until expiry. The base ~4.5% applies whether you lock or not.

#### What is the Merge System?

If you have two NFTs of the same rarity, you can **merge** them into one NFT of the next rarity tier. Both originals are burned and a new one is minted. Higher rarity means a bigger `rarityMultiplier` and a bigger slice of every boost push. Because merging burns NFTs, the collection becomes **more scarce over time**, making remaining NFTs increasingly valuable.

#### How do I sell my XDC NFT?

List or auction your XDC NFT on [**PrimePort.xyz**](https://primeport.xyz). When purchased, the buyer gains the NFT itself plus the full staking position attached to it: staked psXDC shares, pending boost, weight, and any lock status all travel with the NFT.

#### I still hold a legacy XDC NFT. What should I do?

You can keep it (the legacy contracts remain operational) or migrate it to V3 in a single transaction via [`/xdc-nfts/migrate`](https://primestaking.xyz/xdc-nfts/migrate). Migration preserves your **rarity** and any active **lock expiry**, and your **tokenId** for legacy ids below `10000` (ids ≥ `10000` are remapped into the `5558–9999` band because that range is reserved for merged NFTs). It immediately starts earning under the V3 reward model.

→ [Migrate XDC NFTs to V3](xdc-staking-nfts/migrate-nfts-v2-to-v3.md) → [Locked NFTs & Legacy Diamond Bypass](xdc-staking-nfts/locked-nft-migration.md)

***

### XDC Liquid Staking

#### How does Liquid Staking work?

1. **Stake XDC** through the V3 vault (`stake()` payable, or `depositNative(assets, receiver)`).
2. **Receive psXDC shares** at the current exchange rate.
3. **Watch the share price grow.** Rewards are embedded in the share. No claim needed.
4. **Withdraw** by calling `redeemWithQueue`: instant when the buffer covers it, queued FIFO otherwise. Or swap on a DEX for an immediate market exit (verify the pool holds the live V3.1 token).

#### Who can participate?

Anyone. There is **no minimum XDC required**. It's accessible to all XDC holders.

#### How do I withdraw my staked XDC?

You have two options:

* **Protocol redemption** (`redeemWithQueue`): burns the equivalent psXDC shares. **Instant** when the vault's liquid buffer permits, otherwise enters the on-chain **FIFO queue** with self-claim via `claimQueuedAssets`. You can cancel queued requests at any time before settlement.
* **Instant DEX exit**: swap psXDC for XDC on a DEX (verify the pool holds the live V3.1 token) for immediate liquidity at market price.

#### Can I transfer my Liquid Staking position?

Yes. Send psXDC to another address; whoever holds the share owns the appreciating position. No NFT required.

#### I still hold V2 psXDC. How do I get V3?

Use the [V2 → V3 migration bridge](xdc-liquid-staking/staking-guide/migration.md). Approve [`PrimeStakedXDC_V3MigrationBridge`](contract-addresses.md), call `migrate(amount, minSharesOut)`, and your V2 tokens are burned and V3 shares minted in the same transaction with slippage protection.
