# Architecture Overview

PrimeStaking V3 runs entirely on audited, on-chain smart contracts deployed on the XDC Network. The architecture is split into two non-upgradeable cores (the psXDC vault and the migration bridge) plus an upgradeable NFT-staking layer with namespaced storage.

Infrastructure is developed in collaboration with **Nethermind** (smart contract engineering and security) and the **XDC Core team** (network-level validator integration).

---

## System Design

```
                          User / Partner API
                                    │
                                    ▼
                    ┌──────────────────────────────┐
                    │     Frontend / SDK / API     │
                    └──────────────┬───────────────┘
                                   │
        ┌──────────────────────────┼───────────────────────────┐
        │                          │                           │
        ▼                          ▼                           ▼
 ┌──────────────┐         ┌──────────────────┐         ┌──────────────┐
 │ PrimeStaked  │         │  XdcNftStaking   │         │  V3 Migration │
 │ XDC_V3       │◄────────┤  Vault (proxy)   │         │  Bridge       │
 │ (ERC-4626,   │  shares │  + XdcStakedNFT  │         │  (V2 → V3)    │
 │  immutable)  │         │  + Migrator      │         │               │
 └──────┬───────┘         │  + Harvester     │         └──────┬────────┘
        │                 │  + Bypass facet  │                │
        │ stake / redeem  └────────┬─────────┘                │ burns V2 psXDC
        ▼                          │                          ▼
 ┌──────────────────────────────────────────────────────────────────────┐
 │  XDC Network masternodes (on-chain validator contract)               │
 └──────────────────────────────────────────────────────────────────────┘
```

---

## Contract Topology

| Contract | Function | Upgradeability | Address |
| --- | --- | --- | --- |
| `PrimeStakedXDC_V3_2` | ERC-4626 native-XDC vault. Mints/burns psXDC shares, manages liquidity buffer, processes withdrawals, interfaces with masternodes. | **None** (regular constructor, no proxy) | [`0xa7FD…73e4`](https://xdcscan.com/address/0xDc74c0DaED82ae94486DeeF22991d2F54173c734) |
| `PrimeStakedXDC_V3MigrationBridge` | One-way V2 psXDC → V3 share migration. Time-locked admin, daily withdrawal caps. | **None** | [`0x6c57…373C`](https://xdcscan.com/address/0x6c57075c7A157113D369109B78738A798d41373C) |
| `XdcStakedNFT` | ERC-721 NFT collection for staking positions. Rarity stored on-chain. | **None** | [`0xf3eB…898E`](https://xdcscan.com/address/0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E) |
| `XdcNftStakingVault` | Holds psXDC v3 shares per NFT; runs Synthetix-style boost accumulator; handles stake/withdraw/claim/lock/merge/burnAndRedeem. | **TransparentUpgradeableProxy** (ERC-7201 namespaced storage) | [`0x9f38…4Da8`](https://xdcscan.com/address/0x9f38dF64eeC71e2408B24217b8D621c6B07E4Da8) |
| `XdcNftMigratorV2` | Atomic V2 → V3 NFT migration. Preserves `tokenId`/rarity/lock, and remaps legacy ids ≥ `10000` into the free `5558–9999` band. | **None** | [`0x69DE…2ea8`](https://xdcscan.com/address/0x69DE30161ec0f2e0Dc0649190dB9b93F4c492ea8) |
| `XdcNftBoostHarvester` | Funds the NFT vault's boost accumulator via `notifyBoost`. Only holder of `FEE_ROUTER_ROLE`. | **None** | [`0x6a31…821D`](https://xdcscan.com/address/0x6a319528111E5e50712Fd2D3d2db8323b119821D) |
| `LegacyMigratorBypassFacet` | Diamond facet on the legacy Diamond `0x7a5d…aA17` enabling locked-NFT migration (clears `tokenLocked`; the diamond pays the psXDC). | Facet, added via `diamondCut` | [`0x2786…5e13`](https://xdcscan.com/address/0x2786D8Df1C38c9D4eD642B84c073349b0f0B5e13) |

Full inventory in [Deployed Contracts & Addresses](../xdc-staking/xdc-nfts-staking-system-vaults/contract-addresses.md).

---

## Validator Infrastructure

PrimeStaking operates XDC Network masternodes that generate the underlying staking yield:

- **Validator delegation** is performed by `PrimeStakedXDC_V3_2` directly against the on-chain XDC validator contract. No off-chain custodian.
- **Operator onboarding** is admin-controlled (KYC-verified masternode operators); operator scans are bounded by `operatorScanLimit` to prevent gas-griefing.
- **Auto-propose** runs opportunistically during stake or via `triggerAutoPropose(maxNodes)`. It is **blocked whenever the withdrawal queue has a backlog**, so user redemptions are prioritised over new validator locks.
- **Resignation** returns principal to the vault after the network `candidateWithdrawDelay` (~35 days under typical block times). `reportMasternodeResignPrincipal(operator)` accounts for the returned principal without inflating the reward share.
- **Per-operator tracking** of outstanding principal both globally and per operator (`outstandingValidatorPrincipalByOperator`).
- **No principal-stake slashing.** XDC penalizes underperforming masternodes via temporary exclusion (~2h) and missed rewards, but never burns staked capital.

→ [Custody Model](custody-model.md)

---

## Security Layers

| Layer | Implementation |
| --- | --- |
| **Smart contract audits** | QuillAudits (98.8% on liquid staking) + Nethermind Security (custody / V3 surface) |
| **Permissionless custody** | Validator keys and treasury secured by on-chain contracts |
| **On-chain transparency** | Every stake, queued withdrawal, claim, boost notification, migration, and loss report emits a public event |
| **Non-upgradeable cores** | `PrimeStakedXDC_V3_2`, migration bridge, NFT collection, migrator, harvester, and bypass facet are all deployed with regular constructors |
| **Controlled NFT vault upgrades** | Only `XdcNftStakingVault` is upgradeable, via TransparentUpgradeableProxy controlled by the protocol multisig; storage is ERC-7201 namespaced to remain collision-proof |
| **Reentrancy protection** | OpenZeppelin `ReentrancyGuard` on every state-changing function |
| **Pausable surfaces** | Vault, migrator, and harvester each expose `pause()`/`unpause()` under `PAUSER_ROLE` |
| **Delayed governance** | Every sensitive parameter change (role rotations, loss caps, governance delay itself, ownership transfer) is a schedule → wait → execute flow |
| **Loss caps** | `reportValidatorLoss` is bounded by `maxLossBpsPerReport` and `maxDailyLossBps`, both governed via delayed changes |

---

## Data Flow

### Staking

1. User calls `stake()` or `depositNative(assets, receiver)` on `PrimeStakedXDC_V3_2` with native XDC as `msg.value`.
2. Vault mints psXDC shares at the current exchange rate (`totalAssets / totalShares`).
3. Excess liquidity above the buffer triggers auto-propose if no queue backlog exists; XDC is delegated to a masternode through the XDC validator contract.

### Reward Accrual

1. Validator rewards flow back into the vault.
2. `totalAssets` increases; share supply does not.
3. Exchange rate rises automatically, so every psXDC share is worth more XDC. There is no manual `claim` step for the base layer.

### Withdrawal

1. User calls `redeemWithQueue(shares, receiver)` (or `withdrawWithQueue(assets, ...)`).
2. If `maxRedeem(user) >= shares`, the redemption settles **instantly** in the same transaction.
3. Otherwise the request enters the FIFO queue; shares are escrowed inside the vault. Settlement uses the live exchange rate at processing time.
4. `processWithdrawalQueue(maxRequests)` is permissionless; anyone can push the queue forward.
5. Failed receiver payouts defer into `pendingQueuedAssets`; the user claims later via `claimQueuedAssets`.

### Boost (NFT layer)

1. Treasury / harvester pushes XDC into `XdcNftStakingVault.notifyBoost(amount)`.
2. The vault converts XDC to psXDC v3 shares and increments `rewardPerWeightStored`.
3. Each staked NFT's pending boost grows proportionally to its weight (`stakedShares × (rarityMultiplier + level + lockBonus)`).
4. NFT holder calls `claim(tokenId, unwrap)` from the app to settle their slice in XDC or shares.

### V2 → V3 Migration

1. **psXDC**: user approves the bridge, calls `migrate(amount, minSharesOut)`. Bridge burns V2 → vault mints V3 shares while migration window is open.
2. **NFTs**: user approves the migrator, calls `migrate(tokenId, minSharesOut)`. Migrator pulls legacy NFT → claims pending V2 rewards (best-effort) → calls `migratorPrepareForBurn` on the legacy Diamond if locked → `burnAndRedeem` on the legacy façade → bridges redeemed psXDC into V3 shares → `mintAndStake[Locked]` on the new vault under the same `tokenId`.

---

## Integration Points

| Integration Level | Description | Use Case |
| --- | --- | --- |
| **Frontend** | Embed PrimeStaking widgets or build a custom frontend on top of the contracts | White-label web integration |
| **Smart Contract** | Call `PrimeStakedXDC_V3_2` directly for stake/withdraw, or `migrate` on the bridge. ERC-4626 standard means partner contracts can wrap psXDC as collateral. | Backend / API integration |
| **Data / Reporting** | On-chain event indexing via [`staking-v3-indexer`](https://github.com/PrimeNumbersLabs/staking-v3-indexer) and [`xdc-nft-v3-indexer`](https://github.com/PrimeNumbersLabs/xdc-nft-v3-indexer) for portfolio and settlement reporting | Compliance and reconciliation |

→ [Integration Models](integration-models.md) → [Custody Model](custody-model.md) → [Governance](governance.md)
