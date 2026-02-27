# Staking Reward System

The PRFI Staking Reward System transforms NFTs from static collectibles into yield-bearing assets. By depositing PRFI tokens into NFTs, users activate reward mechanisms tied to each NFT's rarity and level.

---

## Core Concept

Users stake PRFI tokens into their NFTs. The NFT acts as a staking vehicle, generating rewards proportional to:

- The **amount staked**
- The NFT's **rarity tier** (base multiplier)
- The NFT's **level** (added multiplier)

Rewards are calculated and distributed monthly.

---

## Features

| Feature | Description |
| --- | --- |
| **Token-to-NFT Staking** | Deposit PRFI into NFTs to activate yield based on rarity and level. |
| **Level Progression** | NFTs progress through levels 1–20 as stake increases, boosting the added multiplier. |
| **Reward Accumulation** | NFTs earn PRFI from the monthly reward pool, royalties, and PrimeFi profits. |
| **Omnichain Support** | NFTs can be bridged between Base and other supported chains via LayerZero. |
| **Merge System** | Combine two same-rarity NFTs into a higher-rarity NFT. |

---

## Benefits

- **Active financial utility** - NFTs generate income, not just sit in a wallet.
- **Scalable returns** - Rewards scale with staking input, level, and rarity.
- **Transparent distribution** - All reward logic is on-chain and auditable.
- **Secure architecture** - Built on audited, upgradeable smart contracts (Diamond Standard / EIP-2535).

---

## Technical Architecture

The staking system is built on a **modular, upgradeable architecture (EIP-2535 Diamond Standard)** with cross-chain capabilities powered by LayerZero:

| Component | Purpose |
| --- | --- |
| `ERC721Facet` | NFT logic: minting, burning, bridging, metadata |
| `StakerFacet` | Staking engine: stake, merge, lock, rewards, claiming |
| `AdminFacet` | Configuration, pausing, access control |
| `GetterFacet` | Read-only data for frontend and integrations |
