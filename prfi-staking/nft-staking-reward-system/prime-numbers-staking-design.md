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

# Staking Design

## Prime Numbers Staking Design

The PRFI NFT Staking System (ONFT) introduces a fully upgradeable and omnichain staking architecture utilizing the Diamond Standard (EIP-2535) and the LayerZero protocol. This allows users to stake, level up, and earn rewards across chains without compromising on transparency, precision, or security.

### Diamond Architecture Overview

All staking logic and NFT management are modularized into _facets_ within a single proxy contract on the main chain:

| Facet         | Description                                                               |
| ------------- | ------------------------------------------------------------------------- |
| `ERC721Facet` | Core NFT logic: minting, burning, bridging, and metadata management       |
| `StakerFacet` | Main staking engine: handles staking, merging, locking, rewards, claiming |
| `AdminFacet`  | Controls configurations, pausing, and authorized addresses                |
| `GetterFacet` | Exposes view-only data for frontend and integrations                      |

### Omnichain NFT Support (ONFT)

NFTs are **omnichain**: they can be moved between Base and other supported chains using LayerZero. A sidechain contract manages local NFT ownership and forwards user actions (stake, claim, withdraw, redeem) to the mainchain. The mainchain executes core staking logic and replies with confirmations or reverts.

Key features:

* All economic logic lives on the mainnet (single source of truth)
* Users interact from low-gas chains (e.g. Base)
* NFTs preserve full metadata and reward history when bridged

### Staking Mechanics

* **Stake PRFI into your NFT** to begin earning rewards
* **NFT Level** increases as stake crosses defined thresholds (up to level 20)
* **Rarity Multiplier** and **Level** combine into a **weight** that determines rewards

#### Reward Calculation

Rewards are distributed monthly and split into two buckets:

* 50% based on the NFT’s `multiplier` (rarity + level)
* 50% based on the NFT’s `stake × weight`

This ensures fair rewards for both quality (rarity) and quantity (stake commitment).

#### Claiming Rewards

* Rewards are claimable on any chain
* Calling `claim()` syncs missed airdrops and updates `lastClaimedAmount`
* High-precision math (`ABDKMathQuad`) ensures exact distribution

#### Merging NFTs

* Two NFTs of the same rarity can be merged into one with a higher rarity
* The resulting NFT inherits combined stake and upgraded multiplier
* Merged NFTs are burned; new NFTs minted from a reserved ID range

### Withdrawing & Redeeming

* Users may **withdraw a portion** of stake (20% fee)
* Once an NFT is no longer needed, users can **burn and redeem**
* Withdrawals are blocked during lock periods or vesting

### Security Highlights

* **Upgradeable via Diamond Standard**: facets can be added or upgraded without migrating contracts
* **Reentrancy Guard**: prevents reentrancy attacks
* **Pausable**: authorized roles can pause contract functions during incidents
* **Only Authorized Messages**: LayerZero endpoints are restricted to trusted chains/contracts
* **All data is on-chain**, auditable, and fully deterministic

### Technical Details

* **Withdrawal Fee**: 20% by default (configurable via AdminFacet)
* **Storage**: NFT state (stake, lock, rewards) is replicated between chains via bridging
* **Rewards Pool**: Funded via protocol earnings, royalties, and monthly airdrops
