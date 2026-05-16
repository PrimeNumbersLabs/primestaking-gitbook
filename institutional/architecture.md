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
| `PrimeStakedXDC_V3` | ERC-4626 native-XDC vault. Mints/burns psXDC shares, manages liquidity buffer, processes withdrawals, interfaces with masternodes. | **None** — regular constructor, no proxy | [`0x98D9…C4Ba`](https://xdcscan.com/address/0x98D916F5773Ac0482b49856f2659d6c32114C4Ba) |
| `PrimeStakedXDC_V3MigrationBridge` | One-way V2 psXDC → V3 share migration. Time-locked admin, daily withdrawal caps. | **None** | [`0x2927…5dD2`](https://xdcscan.com/address/0x2927630dfDd66433DbA9370b316EF5a8408d5dD2) |
| `XdcStakedNFT` | ERC-721 NFT collection for staking positions. Rarity stored on-chain. | **None** | [`0xf3eB…898E`](https://xdcscan.com/address/0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E) |
| `XdcNftStakingVault` | Holds psXDC v3 shares per NFT; runs Synthetix-style boost accumulator; handles stake/withdraw/claim/lock/merge/burnAndRedeem. | **TransparentUpgradeableProxy** (ERC-7201 namespaced storage) | [`0x9f38…4Da8`](https://xdcscan.com/address/0x9f38dF64eeC71e2408B24217b8D621c6B07E4Da8) |
| `XdcNftMigrator` | Atomic V2 → V3 NFT migration with `tokenId`/rarity/lock preservation. | **None** | [`0x45e2…7dFb`](https://xdcscan.com/address/0x45e2e91098A8451EA450754784e043bb3F8C7dFb) |
| `XdcNftBoostHarvester` | Funds the NFT vault's boost accumulator via `notifyBoost`. Only holder of `FEE_ROUTER_ROLE`. | **None** | [`0x3bEd…8DeA`](https://xdcscan.com/address/0x3bEdb37FC873F64BEeFCA551b3A836e59fc18DeA) |
| `LegacyMigratorBypassFacet` | Diamond facet on the legacy Diamond `0x7a5d…aA17` enabling locked-NFT migration. | Facet — added via `diamondCut` | [`0x2756…1836`](https://xdcscan.com/address/0x275641d5bA81786A7e60352F990F0c203e7D1836) |

Full inventory in [Deployed Contracts & Addresses](../xdc-staking/xdc-nfts-staking-system-vaults/contract-addresses.md).

---

## Validator Infrastructure

PrimeStaking operates XDC Network masternodes that generate the underlying staking yield:

- **Validator delegation** is performed by `PrimeStakedXDC_V3` directly against the on-chain XDC validator contract. No off-chain custodian.
- **Operator onboarding** is admin-controlled (KYC-verified masternode operators); operator scans are bounded by `operatorScanLimit` to prevent gas-griefing.
- **Auto-propose** runs opportunistically during stake or via `triggerAutoPropose(maxNodes)`. It is **blocked whenever the withdrawal queue has a backlog** — user redemptions are prioritised over new validator locks.
- **Resignation** returns principal to the vault after the network `candidateWithdrawDelay` (~35 days under typical block times). `reportMasternodeResignPrincipal(operator)` accounts for the returned principal without inflating the reward share.
- **Per-operator tracking** of outstanding principal both globally and per operator (`outstandingValidatorPrincipalByOperator`).
- **Zero slashing risk** — XDC Network's masternode model has no slashing.

→ [Custody Model](custody-model.md)

---

## Security Layers

| Layer | Implementation |
| --- | --- |
| **Smart contract audits** | QuillAudits (98.8% on liquid staking) + Nethermind Security (custody / V3 surface) |
| **Permissionless custody** | Validator keys and treasury secured by on-chain contracts |
| **On-chain transparency** | Every stake, queued withdrawal, claim, boost notification, migration, and loss report emits a public event |
| **Non-upgradeable cores** | `PrimeStakedXDC_V3`, migration bridge, NFT collection, migrator, harvester, and bypass facet are all deployed with regular constructors |
| **Controlled NFT vault upgrades** | Only `XdcNftStakingVault` is upgradeable, via TransparentUpgradeableProxy controlled by the protocol multisig; storage is ERC-7201 namespaced to remain collision-proof |
| **Reentrancy protection** | OpenZeppelin `ReentrancyGuard` on every state-changing function |
| **Pausable surfaces** | Vault, migrator, and harvester each expose `pause()`/`unpause()` under `PAUSER_ROLE` |
| **Delayed governance** | Every sensitive parameter change (role rotations, loss caps, governance delay itself, ownership transfer) is a schedule → wait → execute flow |
| **Loss caps** | `reportValidatorLoss` is bounded by `maxLossBpsPerReport` and `maxDailyLossBps`, both governed via delayed changes |

---

## Data Flow

### Staking

1. User calls `stake()` or `depositNative(assets, receiver)` on `PrimeStakedXDC_V3` with native XDC as `msg.value`.
2. Vault mints psXDC shares at the current exchange rate (`totalAssets / totalShares`).
3. Excess liquidity above the buffer triggers auto-propose if no queue backlog exists; XDC is delegated to a masternode through the XDC validator contract.

### Reward Accrual

1. Validator rewards flow back into the vault.
2. `totalAssets` increases; share supply does not.
3. Exchange rate rises automatically — every psXDC share is worth more XDC. There is no manual `claim` step for the base layer.

### Withdrawal

1. User calls `redeemWithQueue(shares, receiver)` (or `withdrawWithQueue(assets, ...)`).
2. If `maxRedeem(user) >= shares`, the redemption settles **instantly** in the same transaction.
3. Otherwise the request enters the FIFO queue; shares are escrowed inside the vault. Settlement uses the live exchange rate at processing time.
4. `processWithdrawalQueue(maxRequests)` is permissionless — anyone can push the queue forward.
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
| **Smart Contract** | Call `PrimeStakedXDC_V3` directly for stake/withdraw, or `migrate` on the bridge. ERC-4626 standard means partner contracts can wrap psXDC as collateral. | Backend / API integration |
| **Data / Reporting** | On-chain event indexing via [`staking-v3-indexer`](https://github.com/PrimeNumbersLabs/staking-v3-indexer) and [`xdc-nft-v3-indexer`](https://github.com/PrimeNumbersLabs/xdc-nft-v3-indexer) for portfolio and settlement reporting | Compliance and reconciliation |

→ [Integration Models](integration-models.md) → [Custody Model](custody-model.md) → [Governance](governance.md)
