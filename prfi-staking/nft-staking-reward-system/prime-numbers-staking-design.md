# Staking Design

The PRFI NFT Staking System uses a fully upgradeable, omnichain architecture built on the Diamond Standard (EIP-2535) and LayerZero. Users stake PRFI, level up NFTs, and earn rewards across chains.

---

## Architecture

All staking logic and NFT management are modularized into facets within a single proxy contract on the main chain:

| Facet | Description |
| --- | --- |
| `ERC721Facet` | Core NFT logic: minting, burning, bridging, metadata |
| `StakerFacet` | Staking engine: stake, merge, lock, rewards, claiming |
| `AdminFacet` | Configuration, pausing, access control |
| `GetterFacet` | Read-only data for frontend and integrations |

---

## Omnichain NFT Support (ONFT)

NFTs are **omnichain** — they can be bridged between Base and other supported chains using LayerZero.

- All economic logic lives on the mainnet (single source of truth).
- Users interact from low-gas chains (e.g., Base).
- NFTs preserve full metadata and reward history when bridged.

A sidechain contract manages local NFT ownership and forwards user actions (stake, claim, withdraw, redeem) to the mainchain. The mainchain executes the logic and replies with confirmations.

---

## Staking Mechanics

- **Stake PRFI into your NFT** to begin earning rewards.
- **NFT Level** increases as stake crosses defined thresholds (up to level 20).
- **Rarity Multiplier** and **Level** combine into a **weight** that determines rewards.

### Reward Calculation

Rewards are distributed monthly and split into two equal buckets:

- **50%** based on the NFT's `multiplier` (rarity + level)
- **50%** based on `stake x weight` (multiplier x staked PRFI)

This ensures fair rewards for both quality (rarity) and quantity (stake commitment).

### Claiming

- Rewards are claimable on any chain.
- Calling `claim()` syncs missed distributions and updates the last claimed amount.
- High-precision math (`ABDKMathQuad`) ensures exact distribution.

### Merging

- Two NFTs of the same rarity can be merged into one with a higher rarity.
- The resulting NFT inherits the combined stake and an upgraded multiplier.
- Merged NFTs are burned; a new NFT is minted from a reserved ID range.

---

## Withdrawing & Redeeming

- **Partial withdrawal:** Remove a portion of staked PRFI (20% fee, redistributed to other holders).
- **Burn to redeem:** Destroy the NFT to withdraw all staked PRFI.
- Withdrawals are blocked during lock periods.

---

## Security

| Measure | Detail |
| --- | --- |
| **Diamond Standard** | Facets can be added or upgraded without contract migration |
| **Reentrancy Guard** | Prevents reentrancy attacks |
| **Pausable** | Authorized roles can pause functions during incidents |
| **Trusted Endpoints** | LayerZero messages restricted to trusted chains/contracts |
| **On-chain** | All data is auditable and fully deterministic |

---

## Technical Parameters

| Parameter | Value |
| --- | --- |
| Withdrawal fee | 20% (configurable) |
| Max level | 20 |
| Rewards pool | Funded via protocol earnings, royalties, and monthly airdrops |
| Storage | NFT state replicated between chains via bridging |
