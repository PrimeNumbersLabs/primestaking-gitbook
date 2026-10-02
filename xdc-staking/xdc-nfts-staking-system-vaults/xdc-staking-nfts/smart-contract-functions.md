# Smart Contract Reference (V3)

Technical reference for the V3 XDC NFT stack. There are five distinct contracts; users only interact with the **`XdcNftStakingVault`** proxy and (during the migration window) the **`XdcNftMigrator`**.

| Contract | Address | Type |
| --- | --- | --- |
| `XdcStakedNFT` | [`0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E`](https://xdcscan.com/address/0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E) | ERC-721 collection, non-upgradeable |
| `XdcNftStakingVault` (proxy) | [`0x9f38dF64eeC71e2408B24217b8D621c6B07E4Da8`](https://xdcscan.com/address/0x9f38dF64eeC71e2408B24217b8D621c6B07E4Da8) | TransparentUpgradeableProxy, ERC-7201 storage |
| `XdcNftMigrator` | [`0x87Abbf807Be90E3c618Db410c6BEd7c1aA38556A`](https://xdcscan.com/address/0x87Abbf807Be90E3c618Db410c6BEd7c1aA38556A) | Current `MIGRATOR_ROLE` holder (since 9 Aug 2026), non-upgradeable. The 1,431-NFT migration itself ran 15 May – 4 Jul 2026 through `0x45e2…7dFb`, `0x36Fe…f026` and `0x69DE…2ea8` (all revoked). |
| `XdcNftBoostHarvester` | [`0x6a319528111E5e50712Fd2D3d2db8323b119821D`](https://xdcscan.com/address/0x6a319528111E5e50712Fd2D3d2db8323b119821D) | Optional boost feeder (native XDC lanes only), non-upgradeable |
| `LegacyMigratorBypassFacet` | [`0x2786D8Df1C38c9D4eD642B84c073349b0f0B5e13`](https://xdcscan.com/address/0x2786D8Df1C38c9D4eD642B84c073349b0f0B5e13) | Facet added to legacy Diamond `0x7a5d…aA17` |

---

## `XdcNftStakingVault`: the staking engine

### User functions

| Function | What it does |
| --- | --- |
| `stake(uint256 tokenId, uint256 shares)` | Pulls `shares` of psXDC v3 from `msg.sender` and stakes them against `tokenId`. Settles pending boost first. Reverts `ExceedsMaxStakePerNft` if the resulting balance would exceed `maxStakePerNft` (default 100,000 psXDC). |
| `withdraw(uint256 tokenId, uint256 shares)` | Returns `shares` of psXDC v3 from the NFT to `msg.sender`. Reverts if the NFT is locked. |
| `claim(uint256 tokenId, bool unwrap)` | Pays out the NFT's earned boost. If `unwrap == true`, redeems the boost shares to native XDC; otherwise transfers shares. |
| `lock(uint256 tokenId, uint64 duration)` | Locks the NFT for one of the configured tiers (30, 90, 180 or 365 days → +1, +3, +6 or +12 boost units) and freezes those units into its weight. Disables `withdraw`/`merge`/`burnAndRedeem` until `lockEnd`. |
| `pokeExpired(uint256[] tokenIds)` | Permissionless. Clears the lock and boost of NFTs whose `lockEnd` has passed (the boost stops counting at `lockEnd` either way; this settles the bookkeeping). Skips ids that are not expired. |
| `merge(uint256 tokenIdA, uint256 tokenIdB)` | Burns two same-rarity NFTs, mints one higher-rarity NFT via `XdcStakedNFT.mintMerged`, settles boost on both. Reverts `ExceedsMaxStakePerNft` if the two NFTs' combined shares would exceed `maxStakePerNft`. |
| `burnAndRedeem(uint256 tokenId, bool unwrap)` | Burns the NFT and returns the underlying shares (or unwraps them to XDC) in one transaction. |
| `notifyBoost(uint256 amount) payable` | **`FEE_ROUTER_ROLE` only** (harvester and operations wallet). Receives `amount` native XDC, mints psXDC v3 shares, bumps `rewardPerWeightStored`. Reverts if `totalWeight == 0`. |
| `notifyBoostShares(uint256 shares)` | **`FEE_ROUTER_ROLE` only.** Added 14 Sep 2026. Pulls `shares` psXDC from the caller (needs prior `approve`) and credits them to the accumulator exactly like `notifyBoost`, without any native XDC — lets the operator pay the boost from psXDC holdings and leaves the psXDC vault's liquidity buffer untouched. Emits `BoostNotified(0, shares, delta)`. Reverts on zero shares or `totalWeight == 0`. |

### Migrator-only functions

| Function | Role | What it does |
| --- | --- | --- |
| `mintAndStake(address to, uint256 tokenId, uint8 rarity, uint256 shares)` | `MIGRATOR_ROLE` | Mints `tokenId` on the collection with the given rarity and immediately stakes `shares` against it for `to`. |
| `mintAndStakeLocked(address to, uint256 tokenId, uint8 rarity, uint256 shares, uint64 lockEnd, uint64 lockBoostUnits)` | `MIGRATOR_ROLE` | Same, but preserves the legacy NFT's unlock date. The boost units come from the migrator's `migratorLockBoostUnits`: 0 on every migrator until 2 Oct 2026, 12 (the 365-day tier) since. |

### Read-only helpers

| Function | Returns |
| --- | --- |
| `nftState(uint256 tokenId)` | Full state bundle: rarity, stakedShares, level, lockEnd, lockBoost, rewardIndex, weight |
| `earned(uint256 tokenId)` | Pending boost earned by the NFT (not yet claimed) |
| `totalWeight()` | Global weight across every staked NFT |
| `rewardPerWeightStored()` | The Synthetix accumulator's running total |
| `maxStakePerNft()` | Per-NFT staked-shares cap in wei (`0` = unlimited). Default 100,000 psXDC = `100000e18`. |
| `VAULT_STORAGE_SLOT()` | ERC-7201 namespaced storage slot (constant, for upgrade verification) |

### Admin

- `pause()` / `unpause()`: `PAUSER_ROLE`. Halts stake/withdraw/claim; boost can still be received.
- `recoverOrphanedShares(uint256 tokenId, address to)`: `DEFAULT_ADMIN_ROLE`, only `whenPaused` and only for burned NFTs.
- `setLevelStakedNeeded(...)` / `setLockBoost(...)`: only callable while `totalWeight == 0`. `setLockBoostPostLaunch(duration, units)` (`DEFAULT_ADMIN_ROLE`) enables or changes a tier later; it only affects future locks.
- `adminUnlock(uint256 tokenId)`: `DEFAULT_ADMIN_ROLE`. Clears an NFT's lock and its boost.
- `adminSetLockBoost(uint256[] tokenIds, uint64 units)`: `DEFAULT_ADMIN_ROLE`. Added 2 Oct 2026. Sets the boost units of NFTs whose lock is still running, capped at the 365-day tier. Settles boost earned so far at the old weight first and emits `Locked`; ids without an active lock are skipped. Used once, to give the 13 still-locked NFTs migrated from V2 their 365-day boost.
- `setMaxStakePerNft(uint256 maxShares)`: `DEFAULT_ADMIN_ROLE`. Sets the per-NFT stake cap (`0` disables it). Settable at any time; only gates future `stake`/`merge` and never touches existing balances (over-cap NFTs are grandfathered). Migrator mint paths are exempt. See [Per-NFT stake cap](xdc-staking-nfts-mechanics.md#per-nft-stake-cap).

---

## `XdcStakedNFT`: the collection

| Function | Role | Purpose |
| --- | --- | --- |
| `mintWithId(address to, uint256 tokenId, uint8 rarity)` | `MINTER_ROLE` (granted to migrator) | Mints a legacy-tokenId NFT. **Reverts `TokenIdOutOfRange` for `tokenId == 0` or `tokenId ≥ 10000`**, since the `≥ 10000` band is reserved for merges. This is exactly why `XdcNftMigratorV2` remaps high legacy ids before minting. |
| `mintMerged(address to, uint8 rarity)` | `MINTER_ROLE` (granted to vault) | Mints a fresh higher-rarity NFT (10000+ range). |
| `burn(uint256 tokenId)` | `MINTER_ROLE` | Used by `merge` and `burnAndRedeem` flows. |
| `setRarityURI(uint8 rarity, string uri)` | `URI_SETTER_ROLE` | Updates the per-rarity `tokenURI` |
| `rarityOf(uint256 tokenId)` | view | Per-token rarity |

The collection is **non-upgradeable**.

---

## `XdcNftMigratorV2`: the V2 → V3 migrator

The migrator that carried most of the migration is **`XdcNftMigratorV2`** (`0x36Fe…f026`, then `0x69DE…2ea8` for the V3.1 wiring); the current `MIGRATOR_ROLE` holder is `0x87Ab…556A`. It is a drop-in successor to the original `XdcNftMigrator` (paused) that adds **legacy-id remapping**. Same `migrate` / `migrateBatch` surface; the only behavioural change is for legacy ids ≥ `10000`.

| Function | Notes |
| --- | --- |
| `migrate(uint256 oldTokenId, uint256 minSharesOut)` | One-shot migration of a single legacy NFT. Caller must `approve(migrator, oldTokenId)` first. |
| `migrateBatch(uint256[] tokenIds, uint256[] minSharesOuts)` | Loop wrapper. `msg.sender` stays the user (audit fix C-3). |
| `legacyDiamond()` | The legacy Diamond address (`0x7a5d…aA17`), required for locked-NFT migration. |
| `oldFacade()` | The legacy ERC-721 façade address (`0x9D45…76a0`). |

**Id remapping.** `XdcStakedNFT.mintWithId` rejects ids ≥ `10000` (reserved for merges), so a legacy NFT minted in that band could never be minted 1:1. `XdcNftMigratorV2` detects `oldTokenId ≥ 10000`, allocates a free id in the `5558–9999` reserve band, mints the v3 NFT under that **new** id, and emits `LegacyIdRemapped(oldTokenId, newTokenId)`. Rarity, staked value, and lock state are preserved; only the numeric id changes, and only for the ~21 affected legacy NFTs. Every legacy id below `10000` is still preserved 1:1.

Locked NFTs revert with `LegacyDiamondRequiredForLockedNft(tokenId)` if `legacyDiamond == address(0)` (i.e. the migrator was deployed before the bypass facet was cut in).

Migration mechanics in detail: [Migrate XDC NFTs to V3](migrate-nfts-v2-to-v3.md). Locked-NFT specifics: [Locked NFTs & Legacy Diamond Bypass](locked-nft-migration.md).

---

## `XdcNftBoostHarvester`: the boost pipe

| Function | Role | Purpose |
| --- | --- | --- |
| `feed(uint256 amount) payable` | treasury | Directly forwards `amount` native XDC into `vault.notifyBoost`. |
| `harvest(uint256 sharesToRedeem)` | treasury | Redeems `sharesToRedeem` of psXDC v3 through `redeemWithQueue`; the resulting XDC is forwarded to `notifyBoost`. |
| `forwardPending()` | anyone | Pushes any XDC sitting in the harvester (e.g. from queued redemption settling) into `notifyBoost`. |

Full design write-up: [Boost Harvester (technical)](boost-harvester.md).

---

## `LegacyMigratorBypassFacet`: diamond facet

Added to the legacy Diamond via `diamondCut`. Only one mutator, only callable by the migrator:

| Function | Caller | Purpose |
| --- | --- | --- |
| `migratorPrepareForBurn(address asset, uint256 tokenId)` | migrator only | Clears the diamond's `tokenLocked` flag so `burnAndRedeem` succeeds on a locked NFT. For `lockedFromV2` NFTs it first enforces the original v2 `unlockTimestamp` guard (a still-active lock cannot escape), then clears the flag. It makes **no** external call: the diamond custodies the psXDC and pays from its own reserve. *(The original facet called `primeV2.burnToRedeem` here; that path was removed because the v2 staker is drained to ~0, so the call reverted and was never needed.)* |
| `isMigratorBypassNeeded(address asset, uint256 tokenId)` | view | Informational. Returns true if the migrator needs to call `migratorPrepareForBurn` before burning. |
| `lockedFromV2UnlockTimestamp(address asset, uint256 tokenId)` | view | Reads the real `lockedFromV2` unlock time from legacy storage. |

The facet reads via `LegacyAppStorageMirror`, which exposes the **actual** storage flag rather than the façade-synthesised view. This matters because the legacy `StakerGetterFacet.getNFTData` view can misreport `lockedFromV2`. See [Locked NFTs & Legacy Diamond Bypass](locked-nft-migration.md) for the full caveat.

---

## Events worth indexing

| Event | Contract | When |
| --- | --- | --- |
| `Staked` / `Withdrawn` / `Claimed` / `Locked` / `Merged` / `BurnedAndRedeemed` | vault | Standard user actions |
| `MaxStakePerNftSet(uint256 maxShares)` | vault | Per-NFT stake cap changed (`0` = disabled) |
| `BoostNotified(uint256 amountIn, uint256 sharesMinted, uint256 rewardPerWeightStored, uint256 totalWeight)` | vault | Each `notifyBoost`; drives boost APR calculation |
| `MintedAndStaked` / `MintedAndStakedLocked` | vault | Migrator created a new NFT |
| `Migrated` / `MigratedLocked` | migrator | One-shot migration completed |
| `LegacyIdRemapped(uint256 oldTokenId, uint256 newTokenId)` | migrator (V2) | A legacy id ≥ `10000` was remapped to a free `5558–9999` id |
| `MigratorBypassPrepared` | legacy diamond (via facet) | Confirms the bypass facet routed the call |

→ [Deployed Contracts & Addresses](../contract-addresses.md) → [Staking Mechanics](xdc-staking-nfts-mechanics.md) → [Reward Model](xdc-nft-staking-reward-system.md)
