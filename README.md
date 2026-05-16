---
description: Non-custodial XDC staking infrastructure. Zero slashing risk.
---

# Overview

{% hint style="info" %}
PrimeStaking has shipped its V3 upgrade. The new **`PrimeStakedXDC_V3`** vault is a fully non-custodial, ERC-4626 share-based design with self-service withdrawals. The XDC NFT stack has also been rebuilt around the V3 vault. V2 contracts remain live for users who have not migrated. See [Migrate V2 psXDC → V3](xdc-staking/xdc-nfts-staking-system-vaults/xdc-liquid-staking/staking-guide/migration.md) and [Migrate XDC NFTs to V3](xdc-staking/xdc-nfts-staking-system-vaults/xdc-staking-nfts/migrate-nfts-v2-to-v3.md).
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

Earn rewards on XDC and PRFI tokens without giving up control of your assets.

### Products

| Product                | Token | Yield                          | How It Works                                                                                       |
| ---------------------- | ----- | ------------------------------ | -------------------------------------------------------------------------------------------------- |
| **XDC Liquid Staking** | XDC   | \~4.5% APY                     | Stake XDC, receive psXDC vault shares. Share price grows as validator rewards accrue.              |
| **XDC NFTs**           | psXDC | ~4.75% (unlocked) → ~6% (locked) | Deposit psXDC shares into NFTs. Earn the underlying NAV plus a rarity- and lock-weighted boost slice. |
| **PRFI NFTs**          | PRFI  | 100K PRFI/mo pool              | Stake PRFI inside NFTs. Rarity determines share.                                                   |

#### XDC Liquid Staking

The simplest way to earn on your XDC. No NFT required. No minimum amount.

1. **Stake XDC** - Deposit any amount into the V3 vault.
2. **Receive psXDC shares** - You receive psXDC at the current vault exchange rate. There is no fixed 1:1 ratio; share price rises as rewards accrue.
3. **Earn rewards** - Rewards are embedded directly in the share price (~4.5% APY). No claim button — your shares simply become worth more XDC over time.
4. **Stay liquid** - psXDC is a standard ERC-20 on the XDC Network. Hold it, transfer it, use it as DeFi collateral, or deposit it into XDC NFTs.
5. **Withdraw anytime** - Burn psXDC shares. If the vault has enough buffer liquidity, you receive XDC instantly in the same transaction. Otherwise your request enters an automatic FIFO queue and you claim once the masternode payouts return.

→ [Learn more about XDC Liquid Staking](xdc-staking/xdc-nfts-staking-system-vaults/xdc-liquid-staking/) → [V3 Architecture](xdc-staking/xdc-nfts-staking-system-vaults/xdc-liquid-staking/v3-architecture.md)

#### XDC NFTs

A gamified staking layer on top of liquid staking. Deposit psXDC shares into collectible NFTs to earn two stacked yields.

* Each NFT has a **rarity** (Plentiful → Handcrafted) that determines its weight in the boost accumulator.
* **Base yield** comes from the underlying psXDC share price growing over time (~4.5% APY).
* **Boost yield** comes from a Synthetix-style accumulator. The protocol's [`XdcNftBoostHarvester`](xdc-staking/xdc-nfts-staking-system-vaults/xdc-staking-nfts/boost-harvester.md) calls `notifyBoost` and the resulting slice is distributed pro-rata to NFT weights.
* **Level up** by merging two same-rarity NFTs into a higher tier.
* **Lock** your NFT to add a lock bonus to its weight (target band ~4.75% unlocked → ~6% locked when boost flow is steady).
* You only claim the **boost slice** from the app — base NAV is automatically inside the shares you get back on withdraw.

→ [Learn more about XDC NFTs](xdc-staking/xdc-nfts-staking-system-vaults/xdc-staking-nfts/) → [Migrate XDC NFTs to V3](xdc-staking/xdc-nfts-staking-system-vaults/xdc-staking-nfts/migrate-nfts-v2-to-v3.md)

#### PRFI NFTs

Staking for PRFI token holders. Deposit PRFI into NFTs to earn a share of the monthly reward pool. PRFI NFTs live on the **Base** network (Ethereum L2) and are tradeable on [OpenSea](https://opensea.io/collection/primenumbers-prfi-onft).

* 100,000 PRFI distributed monthly across all staked NFTs.
* Higher rarity = higher share of the reward pool.
* Merge NFTs to unlock stronger multipliers and maximize yield.

→ [Learn more about PRFI NFTs](prfi-staking/nft-staking-reward-system/prfi-staking-nfts/) → [Buy on OpenSea](https://opensea.io/collection/primenumbers-prfi-onft)

#### How Rewards Reach You

| Product                | What you earn                                    | How you receive it                                                                           |
| ---------------------- | ------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| **XDC Liquid Staking** | Higher psXDC share price                         | Automatic — your shares are worth more XDC over time. No claim button.                       |
| **XDC NFTs**           | Underlying NAV growth + boost slice (in XDC)     | NAV is automatic; **claim boost** from the NFT detail page. Withdraw burns shares back to XDC. |
| **PRFI NFTs**          | PRFI                                             | Monthly distribution, claim from app                                                          |

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

### Two Integration Models

| Model                | Description                                    | Best For                                 |
| -------------------- | ---------------------------------------------- | ---------------------------------------- |
| **White Label**      | Your brand, our infrastructure.                | Exchanges, large custodians              |
| **Powered by Prime** | Co-branded widget. Minimal integration effort. | Wallets, aggregators, regional platforms |

### Why Partners Choose PrimeStaking

* **Zero slashing risk** - XDC Network does not implement slashing, unlike ETH-based protocols.
* **ERC-4626 vault** - psXDC v3 conforms to the same standard used by major DeFi protocols, making integration straightforward.
* **Non-custodial** - smart contract-based validator custody, no third-party key management, no admin mint or `ownerWithdraw`.
* **Audited infrastructure** - QuillAudits (98.8%) on the liquid staking contracts; Nethermind Security covering the custody and V3 surface.
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

* `PrimeStakedXDC_V3` is **non-upgradeable** — deployed with a regular constructor, no proxy. The vault logic cannot be modified after deployment.
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
| PRFI NFTs               | [primestaking.xyz/prfi-nfts/overview](https://primestaking.xyz/prfi-nfts/overview)                   |
| Institutional Solutions | [Institutional & Exchange Solutions](institutional/)                                                 |
| psXDC V3 vault          | [XDCScan](https://xdcscan.com/address/0x98D916F5773Ac0482b49856f2659d6c32114C4Ba)                    |
| Deployed addresses      | [Contract Addresses](xdc-staking/xdc-nfts-staking-system-vaults/contract-addresses.md)               |
| Contact                 | [admin@primenumbers.xyz](mailto:admin@primenumbers.xyz)                                              |
