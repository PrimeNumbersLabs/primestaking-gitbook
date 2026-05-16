# Smart Contract Reference (V3)

Technical reference for the V3 XDC NFT stack. There are five distinct contracts; users only interact with the **`XdcNftStakingVault`** proxy and (during the migration window) the **`XdcNftMigrator`**.

| Contract | Address | Type |
| --- | --- | --- |
| `XdcStakedNFT` | [`0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E`](https://xdcscan.com/address/0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E) | ERC-721 collection, non-upgradeable |
| `XdcNftStakingVault` (proxy) | [`0x9f38dF64eeC71e2408B24217b8D621c6B07E4Da8`](https://xdcscan.com/address/0x9f38dF64eeC71e2408B24217b8D621c6B07E4Da8) | TransparentUpgradeableProxy, ERC-7201 storage |
| `XdcNftMigrator` | [`0x45e2e91098A8451EA450754784e043bb3F8C7dFb`](https://xdcscan.com/address/0x45e2e91098A8451EA450754784e043bb3F8C7dFb) | One-shot migrator, non-upgradeable |
| `XdcNftBoostHarvester` | [`0x3bEdb37FC873F64BEeFCA551b3A836e59fc18DeA`](https://xdcscan.com/address/0x3bEdb37FC873F64BEeFCA551b3A836e59fc18DeA) | Boost feeder, non-upgradeable |
| `LegacyMigratorBypassFacet` | [`0x275641d5bA81786A7e60352F990F0c203e7D1836`](https://xdcscan.com/address/0x275641d5bA81786A7e60352F990F0c203e7D1836) | Facet added to legacy Diamond `0x7a5d…aA17` |

---

## `XdcNftStakingVault` — the staking engine

### User functions

| Function | What it does |
| --- | --- |
| `stake(uint256 tokenId, uint256 shares)` | Pulls `shares` of psXDC v3 from `msg.sender` and stakes them against `tokenId`. Settles pending boost first. |
| `withdraw(uint256 tokenId, uint256 shares)` | Returns `shares` of psXDC v3 from the NFT to `msg.sender`. Reverts if the NFT is locked. |
| `claim(uint256 tokenId, bool unwrap)` | Pays out the NFT's earned boost. If `unwrap == true`, redeems the boost shares to native XDC; otherwise transfers shares. |
| `lock(uint256 tokenId, uint64 until)` | Sets `lockEnd`, adds `lockBonus` to the NFT's weight. Disables `withdraw`/`merge`/`burnAndRedeem`. |
| `unlock(uint256 tokenId)` | Removes `lockBonus` once `lockEnd` has passed. |
| `merge(uint256 tokenIdA, uint256 tokenIdB)` | Burns two same-rarity NFTs, mints one higher-rarity NFT via `XdcStakedNFT.mintMerged`, settles boost on both. |
| `burnAndRedeem(uint256 tokenId, bool unwrap)` | Burns the NFT and returns the underlying shares (or unwraps them to XDC) in one transaction. |
| `notifyBoost(uint256 amount) payable` | **`FEE_ROUTER_ROLE` only** (granted to the harvester). Receives `amount` native XDC, mints psXDC v3 shares, bumps `rewardPerWeightStored`. Reverts if `totalWeight == 0`. |

### Migrator-only functions

| Function | Role | What it does |
| --- | --- | --- |
| `mintAndStake(address to, uint256 tokenId, uint8 rarity, uint256 shares)` | `MIGRATOR_ROLE` | Mints `tokenId` on the collection with the given rarity and immediately stakes `shares` against it for `to`. |
| `mintAndStakeLocked(address to, uint256 tokenId, uint8 rarity, uint256 shares, uint64 lockEnd, uint256 lockBoost)` | `MIGRATOR_ROLE` | Same, but preserves the legacy NFT's lock state. |

### Read-only helpers

| Function | Returns |
| --- | --- |
| `nftState(uint256 tokenId)` | Full state bundle: rarity, stakedShares, level, lockEnd, lockBoost, rewardIndex, weight |
| `earned(uint256 tokenId)` | Pending boost earned by the NFT (not yet claimed) |
| `totalWeight()` | Global weight across every staked NFT |
| `rewardPerWeightStored()` | The Synthetix accumulator's running total |
| `VAULT_STORAGE_SLOT()` | ERC-7201 namespaced storage slot (constant, for upgrade verification) |

### Admin

- `pause()` / `unpause()` — `PAUSER_ROLE`. Halts stake/withdraw/claim; boost can still be received.
- `recoverOrphanedShares(uint256 tokenId, address to)` — `DEFAULT_ADMIN_ROLE`, only `whenPaused` and only for burned NFTs.
- `setLevelStakedNeeded(...)` / `setLockBoost(...)` — only callable while `totalWeight == 0`.

---

## `XdcStakedNFT` — the collection

| Function | Role | Purpose |
| --- | --- | --- |
| `mintWithId(address to, uint256 tokenId, uint8 rarity)` | `MINTER_ROLE` (granted to migrator) | Mints a legacy-tokenId NFT (1–9999 range). |
| `mintMerged(address to, uint8 rarity)` | `MINTER_ROLE` (granted to vault) | Mints a fresh higher-rarity NFT (10000+ range). |
| `burn(uint256 tokenId)` | `MINTER_ROLE` | Used by `merge` and `burnAndRedeem` flows. |
| `setRarityURI(uint8 rarity, string uri)` | `URI_SETTER_ROLE` | Updates the per-rarity `tokenURI` |
| `rarityOf(uint256 tokenId)` | view | Per-token rarity |

The collection is **non-upgradeable**.

---

## `XdcNftMigrator` — the V2 → V3 migrator

| Function | Notes |
| --- | --- |
| `migrate(uint256 oldTokenId, uint256 minSharesOut)` | One-shot migration of a single legacy NFT. Caller must `approve(migrator, oldTokenId)` first. |
| `migrateBatch(uint256[] tokenIds, uint256[] minSharesOuts)` | Loop wrapper. `msg.sender` stays the user (audit fix C-3). |
| `legacyDiamond()` | The legacy Diamond address (`0x7a5d…aA17`) — required for locked-NFT migration. |
| `oldFacade()` | The legacy ERC-721 façade address (`0x9D45…76a0`). |

Locked NFTs revert with `LegacyDiamondRequiredForLockedNft(tokenId)` if `legacyDiamond == address(0)` (i.e. the migrator was deployed before the bypass facet was cut in).

Migration mechanics in detail: [Migrate XDC NFTs to V3](migrate-nfts-v2-to-v3.md). Locked-NFT specifics: [Locked NFTs & Legacy Diamond Bypass](locked-nft-migration.md).

---

## `XdcNftBoostHarvester` — the boost pipe

| Function | Role | Purpose |
| --- | --- | --- |
| `feed(uint256 amount) payable` | treasury | Directly forwards `amount` native XDC into `vault.notifyBoost`. |
| `harvest(uint256 sharesToRedeem)` | treasury | Redeems `sharesToRedeem` of psXDC v3 through `redeemWithQueue`; the resulting XDC is forwarded to `notifyBoost`. |
| `forwardPending()` | anyone | Pushes any XDC sitting in the harvester (e.g. from queued redemption settling) into `notifyBoost`. |

Full design write-up: [Boost Harvester (technical)](boost-harvester.md).

---

## `LegacyMigratorBypassFacet` — diamond facet

Added to the legacy Diamond via `diamondCut`. Only one mutator, only callable by the migrator:

| Function | Caller | Purpose |
| --- | --- | --- |
| `migratorPrepareForBurn(address asset, uint256 tokenId)` | `XdcNftMigrator` only | Clears `tokenLocked` so `burnAndRedeem` will succeed on a locked NFT; for `lockedFromV2` NFTs, also calls `primeV2.burnToRedeem` to release the V2-side stake. |
| `isMigratorBypassNeeded(address asset, uint256 tokenId)` | view | Informational — true if the migrator needs to call `migratorPrepareForBurn` before burning. |
| `lockedFromV2UnlockTimestamp(address asset, uint256 tokenId)` | view | Reads the real `lockedFromV2` unlock time from legacy storage. |

The facet reads via `LegacyAppStorageMirror`, which exposes the **actual** storage flag rather than the façade-synthesised view — important because the legacy `StakerGetterFacet.getNFTData` view can misreport `lockedFromV2`. See [Locked NFTs & Legacy Diamond Bypass](locked-nft-migration.md) for the full caveat.

---

## Events worth indexing

| Event | Contract | When |
| --- | --- | --- |
| `Staked` / `Withdrawn` / `Claimed` / `Locked` / `Merged` / `BurnedAndRedeemed` | vault | Standard user actions |
| `BoostNotified(uint256 amountIn, uint256 sharesMinted, uint256 rewardPerWeightStored, uint256 totalWeight)` | vault | Each `notifyBoost` — drives boost APR calculation |
| `MintedAndStaked` / `MintedAndStakedLocked` | vault | Migrator created a new NFT |
| `Migrated` / `MigratedLocked` | migrator | One-shot migration completed |
| `MigratorBypassPrepared` | legacy diamond (via facet) | Confirms the bypass facet routed the call |

→ [Deployed Contracts & Addresses](../contract-addresses.md) → [Staking Mechanics](xdc-staking-nfts-mechanics.md) → [Reward Model](xdc-nft-staking-reward-system.md)
