---
description: Non-custodial XDC staking infrastructure. No principal-stake slashing.
---

# Overview

{% hint style="info" %}
PrimeStaking runs on the **`PrimeStakedXDC_V3_2`** vault, a fully non-custodial, ERC-4626 share-based design with self-service withdrawals. In July 2026 the vault was redeployed as V3.2 and **every V3.1 holder's balance was mirrored 1:1 via a snapshot airdrop; no user action was needed**. The XDC NFT vault was repointed to the V3.2 token in the same operation and gained multiple lock tiers. V2 holders who never migrated can still do so through the [migration bridge](xdc-staking/xdc-nfts-staking-system-vaults/xdc-liquid-staking/staking-guide/migration.md); legacy NFTs migrate via [Migrate XDC NFTs to V3](xdc-staking/xdc-nfts-staking-system-vaults/xdc-staking-nfts/migrate-nfts-v2-to-v3.md).
{% endhint %}

## Overview

**PrimeStaking** is a non-custodial staking platform and infrastructure provider on the XDC Network, built in collaboration with **Nethermind** and the **XDC Core team**.

The platform is live at [primestaking.xyz](https://primestaking.xyz).

***

### Who Is This For?

| Audience                    | What You'll Find                                | Go To                                         |
| --------------------------- | ----------------------------------------------- | --------------------------------------------- |
| **Users**                   | Stake XDC, earn rewards, explore NFTs           | [For Users](./#for-users)                     |
| **Partners & Institutions** | Integrate XDC liquid staking into your platform | [For Partners](./#for-partners--institutions) |

***

***

## For Users

Earn rewards on XDC without giving up control of your assets.

### Products

| Product                | Token | Yield                          | How It Works                                                                                       |
| ---------------------- | ----- | ------------------------------ | -------------------------------------------------------------------------------------------------- |
| **XDC Liquid Staking** | XDC   | \~5.5% APY                     | Stake XDC, receive psXDC vault shares. Share price grows as validator rewards accrue.              |
| **XDC NFTs**           | psXDC | ~5.5% base + boost slice → up to ~7% locked | Deposit psXDC shares into NFTs. Earn the underlying NAV plus a rarity- and lock-weighted boost slice. |

#### XDC Liquid Staking

The simplest way to earn on your XDC. No NFT required. No minimum amount.

1. **Stake XDC** - Deposit any amount into the V3 vault.
2. **Receive psXDC shares** - You receive psXDC at the current vault exchange rate. There is no fixed 1:1 ratio; share price rises as rewards accrue.
3. **Earn rewards** - Rewards are embedded directly in the share price (~5.5% APY). There is no claim button; your shares simply become worth more XDC over time. Rewards accrue continuously at any backing level (the V3.2 permanent-ledger model).
4. **Stay liquid** - psXDC is a standard ERC-20 on the XDC Network. Hold it, transfer it, use it as DeFi collateral, or deposit it into XDC NFTs.
5. **Withdraw anytime** - Burn psXDC shares. If the vault has enough unencumbered liquidity, you receive XDC instantly in the same transaction. Otherwise your request enters an automatic FIFO queue and you claim once masternode payouts return.
6. **Refer friends** - Share your invite link; when someone stakes for the first time through it, you earn a share of the protocol fee their staking generates. See [Referral Program](xdc-staking/xdc-nfts-staking-system-vaults/xdc-liquid-staking/referral-program.md).

→ [Learn more about XDC Liquid Staking](xdc-staking/xdc-nfts-staking-system-vaults/xdc-liquid-staking/) → [V3 Architecture](xdc-staking/xdc-nfts-staking-system-vaults/xdc-liquid-staking/v3-architecture.md)

#### XDC NFTs

A gamified staking layer on top of liquid staking. Deposit psXDC shares into collectible NFTs to earn two stacked yields.

* Each NFT has a **rarity** (Plentiful → Handcrafted) that determines its weight in the boost accumulator.
* **Base yield** comes from the underlying psXDC share price growing over time (~5.5% APY).
* **Boost yield** comes from a Synthetix-style accumulator. The protocol's [`XdcNftBoostHarvester`](xdc-staking/xdc-nfts-staking-system-vaults/xdc-staking-nfts/boost-harvester.md) calls `notifyBoost` and the resulting slice is distributed pro-rata to NFT weights.
* **Level up** by merging two same-rarity NFTs into a higher tier.
* **Lock** your NFT for a fixed period to add a lock bonus to its weight. Four tiers are available - **30, 90, 180 or 365 days** - with progressively larger boosts. The boost applies for the whole lock and **ends when the lock expires**; you can re-lock afterwards. Floor is the **~5.5% base NAV** (always earned, no claim); with the boost stream flowing, combined APY ranges from **~5.75% (unlocked)** up to **~7% (365-day lock)**.
* You only claim the **boost slice** from the app. Base NAV is automatically inside the shares you get back on withdraw.

→ [Learn more about XDC NFTs](xdc-staking/xdc-nfts-staking-system-vaults/xdc-staking-nfts/) → [Migrate XDC NFTs to V3](xdc-staking/xdc-nfts-staking-system-vaults/xdc-staking-nfts/migrate-nfts-v2-to-v3.md)

#### How Rewards Reach You

| Product                | What you earn                                    | How you receive it                                                                           |
| ---------------------- | ------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| **XDC Liquid Staking** | Higher psXDC share price                         | Automatic: your shares are worth more XDC over time. No claim button.                       |
| **XDC NFTs**           | Underlying NAV growth + boost slice (in XDC)     | NAV is automatic; **claim boost** from the NFT detail page. Withdraw burns shares back to XDC. |

#### The psXDC Ecosystem

```
XDC  →  psXDC shares  →  NFT  →  Boost slice
```

| Step        | What Happens                                                                                |
| ----------- | ------------------------------------------------------------------------------------------- |
| **Stake**   | Deposit XDC into the V3 vault.                                                              |
| **Receive** | Get psXDC shares at the current exchange rate. NAV starts accruing immediately.             |
| **Use**     | Hold psXDC, trade it on a DEX, or deposit it into an XDC NFT.                               |
| **Boost**   | NFT rarity, level, and lock add weight to your slice of the boost accumulator.              |

***

***

## For Partners & Institutions

PrimeStaking is **XDC liquid staking infrastructure** - built for exchanges, custodians, and institutional partners who want to offer XDC staking to their customers without building from scratch.

### Three Partner Models

| Model                | Description                                                                          | Best For                                 |
| -------------------- | ------------------------------------------------------------------------------------ | ---------------------------------------- |
| **White Label**      | Your brand, our infrastructure.                                                      | Exchanges, large custodians              |
| **Powered by Prime** | Co-branded widget. Minimal integration effort.                                       | Wallets, aggregators, regional platforms |
| **Partner Staking**  | Deploy and self-manage **your own** liquid staking pool, listed in the PrimeStaking app. Flat 15% protocol fee. | Communities, validators, regional platforms |

White Label and Powered by Prime are integration tracks where you embed the flagship PrimeStaking vault (see [For Partners](#for-partners--institutions) below). **Partner Staking** is a distinct, self-service product where you run your own vault (see [Partner Staking](partner-staking/README.md)).

### Why Partners Choose PrimeStaking

* **No principal-stake slashing** - XDC's slashing mechanism penalizes downtime via temporary exclusion from block production (~2h) and missed rewards; principal stake is never burned, unlike ETH-based protocols.
* **ERC-4626 vault** - psXDC v3 conforms to the same standard used by major DeFi protocols, making integration straightforward.
* **Non-custodial** - smart contract-based validator custody, no third-party key management, no admin mint or `ownerWithdraw`.
* **Audited infrastructure** - QuillAudits (98.8%) on the V1 liquid staking contracts; **Nethermind Security NM-0843** (published May 08, 2026) on the psXDC V3 vault and V3 Migration Bridge.
* **Nethermind Security** - trusted auditing partner for Lido, EtherFi, Optimism, and Worldcoin.
* **Battle-tested** - live since 2024 with $6M+ TVL.
* **Revenue sharing** - transparent, on-chain fee model.

### Partner Documentation

| Section                                                   | What It Covers                                             |
| --------------------------------------------------------- | ---------------------------------------------------------- |
| [Institutional Overview](institutional/)                  | What we offer, why PrimeStaking, why XDC Network           |
| [Architecture](institutional/architecture.md)             | System design, contract topology, validator infrastructure |
| [Custody Model](institutional/custody-model.md)           | Permissionless smart contract-based key management         |
| [Integration Models](institutional/integration-models.md) | White Label vs. Powered by Prime                           |
| [Revenue Model](institutional/revenue-model.md)           | Revenue generation, partner sharing, settlement            |
| [Reward Mechanics](institutional/reward-mechanics.md)     | How staking rewards are generated and distributed          |
| [Liquidity Model](institutional/liquidity-model.md)       | psXDC liquidity, share price, redemption mechanics         |
| [Governance](institutional/governance.md)                 | Role separation, delayed governance, what is upgradeable   |
| [Risk & Compliance](institutional/risk-and-compliance.md) | Risk framework, audit history, regulatory posture          |
| [SLA & Support](institutional/sla-and-support.md)         | Uptime commitments, incident response, partner support     |

**Contact:** [admin@primenumbers.xyz](mailto:admin@primenumbers.xyz)

***

***

### Security

* `PrimeStakedXDC_V3_2` is **non-upgradeable**, deployed with a regular constructor and no proxy. The vault logic cannot be modified after deployment. The referral contracts (`ReferralRegistry`, `ReferralRewards`) are likewise non-upgradeable satellites; they never hold user principal.
* The XDC NFT staking vault is a `TransparentUpgradeableProxy` controlled by the protocol multisig, with namespaced ERC-7201 storage. All other NFT-stack contracts (collection, migrator, harvester, bypass facet) are non-upgradeable.
* Validator custody is **smart contract-based** - permissionless and trustless, no third-party custodian.
* The protocol is **non-custodial** - users retain full ownership of their assets at all times.
* Infrastructure developed with **Nethermind** and the **XDC Core team**.

→ [Audits & Security](audits-1/) → [Custody & Key Management](custody-and-key-management.md) → [Deployed Contracts & Addresses](xdc-staking/xdc-nfts-staking-system-vaults/contract-addresses.md)

***

### Quick Links

| Resource                | Link                                                                                                 |
| ----------------------- | ---------------------------------------------------------------------------------------------------- |
| App                     | [primestaking.xyz](https://primestaking.xyz)                                                         |
| XDC Liquid Staking      | [primestaking.xyz/xdc-liquid-staking/overview](https://primestaking.xyz/xdc-liquid-staking/overview) |
| XDC NFTs                | [primestaking.xyz/xdc-nfts/overview](https://primestaking.xyz/xdc-nfts/overview)                     |
| Migrate XDC NFTs        | [primestaking.xyz/xdc-nfts/migrate](https://primestaking.xyz/xdc-nfts/migrate)                       |
| Migrate psXDC V2 → V3   | [primestaking.xyz/xdc-liquid-staking/migration](https://primestaking.xyz/xdc-liquid-staking/migration) |
| Referral Program        | [primestaking.xyz/xdc-liquid-staking/referral](https://primestaking.xyz/xdc-liquid-staking/referral) |
| Institutional Solutions | [Institutional & Exchange Solutions](institutional/)                                                 |
| Partner Staking         | [Run your own staking pool](partner-staking/README.md)                                               |
| psXDC V3.2 vault        | [XDCScan](https://xdcscan.com/address/0xDc74c0DaED82ae94486DeeF22991d2F54173c734)                    |
| Deployed addresses      | [Contract Addresses](xdc-staking/xdc-nfts-staking-system-vaults/contract-addresses.md)               |
| Contact                 | [admin@primenumbers.xyz](mailto:admin@primenumbers.xyz)                                              |
