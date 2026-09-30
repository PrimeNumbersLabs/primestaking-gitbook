# XDC NFTs

XDC Staking NFTs are gamified staking positions on the XDC Network. Each NFT holds **psXDC v3 vault shares** and earns two stacked yields: a **base ~4.5%** from the underlying share-price appreciation (always earned, no claim, no rarity dependency) and an additional **boost slice up to ~1.5%** from a Synthetix-style accumulator funded by the protocol's reward harvester, weighted by rarity, level, and lock status.

{% hint style="info" %}
The XDC NFT stack has been rebuilt around the psXDC v3 vault. New contract addresses: [`XdcStakedNFT`](../contract-addresses.md), [`XdcNftStakingVault`](../contract-addresses.md), [`XdcNftMigrator`](../contract-addresses.md), [`XdcNftBoostHarvester`](../contract-addresses.md), [`LegacyMigratorBypassFacet`](../contract-addresses.md). The legacy V2 collection at `0x9D45…76a0` remains operational for any holder who has not yet migrated; see [Migrate XDC NFTs to V3](migrate-nfts-v2-to-v3.md).
{% endhint %}

---

## Key Facts

| | |
| --- | --- |
| **Collection size** | 5,557 NFTs (5,542 generative + 15 handcrafted), preserved across V2 → V3 |
| **Token staked** | **psXDC v3 vault shares** (not raw XDC) |
| **Max stake per NFT** | **100,000 psXDC** (share count). Stake more by holding more NFTs. See [Per-NFT stake cap](xdc-staking-nfts-mechanics.md#per-nft-stake-cap). |
| **Base yield** | ~4.5%, from share-price appreciation of the underlying psXDC |
| **Boost slice** | Up to ~1.5%: Synthetix accumulator, weighted by rarity / level / lock |
| **Floor APY** | **~4.5%** (the base NAV), automatic, always earned regardless of rarity / lock / boost cadence |
| **Target APY band (with boost)** | ~4.75% (unlocked) → ~6% (locked) when boost stream is steady |
| **Reward token (boost)** | psXDC shares. The boost is funded in XDC (`notifyBoost`, wrapped into shares) or directly in psXDC (`notifyBoostShares`); the app claims it as psXDC (`claim(tokenId, false)`), and the contract can also unwrap to native XDC on request (`unwrap = true`, subject to the vault's liquidity/queue) |
| **Locked yield** | Additive `lockBoost` term added to NFT weight when locked |
| **Merge** | Two same-rarity NFTs → one higher-rarity NFT (originals burned) |
| **Marketplace** | [PrimePort.xyz](https://primeport.xyz) |

---

## How It Works

1. **Get psXDC shares.** Stake XDC in [`PrimeStakedXDC_V3_2`](../xdc-liquid-staking/README.md) or buy psXDC on a DEX. (Already hold V2 psXDC? [Migrate to V3 first](../xdc-liquid-staking/staking-guide/migration.md).)
2. **Get an NFT.** Buy one on [PrimePort](https://primeport.xyz), or migrate a legacy V2 NFT through [`XdcNftMigratorV2`](migrate-nfts-v2-to-v3.md) (preserves your rarity and any active lock, and your `tokenId` for legacy ids below `10000`; ids ≥ `10000` are remapped).
3. **Stake psXDC shares into your NFT.** The vault records the shares against the NFT's `tokenId`. The NFT's **weight** in the boost accumulator becomes `stakedShares × (rarityMultiplier + level + lockBoost)`. Each NFT holds up to **100,000 psXDC**; to stake more, spread it across multiple NFTs.
4. **Earn two stacked yields**:
   - **Base NAV**: your staked shares keep appreciating; you receive them back at the higher value when you withdraw.
   - **Boost**: every boost push (`notifyBoost` in XDC or `notifyBoostShares` in psXDC, from the operations wallet or the harvester) increments `rewardPerWeightStored`; your earned slice grows in proportion to your weight. Pushes are operator-driven batches, not a continuous stream.
5. **Claim boost** from the NFT detail page whenever you want. It is paid in psXDC shares (which you can stake back into the NFT, hold, or redeem). Base NAV is automatic and needs no claim.
6. **Upgrade** by merging two same-rarity NFTs into a higher-tier one for a larger `rarityMultiplier`.
7. **Lock (optional)**: locking adds `lockBoost` to the weight calculation. Lock expiry is preserved across migration so users can't dodge the lock by routing through the migrator.
8. **`burnAndRedeem`** burns the NFT and returns the underlying psXDC shares (or, optionally, redeems them to XDC in one transaction).

---

## Rarity Tiers

Each NFT has a rarity that determines its `rarityMultiplier`, which feeds into the weight formula:

| Rarity | `rarityMultiplier` (weight units) |
| --- | --- |
| Plentiful | 3 |
| Common | 4 |
| Uncommon | 5 |
| Rare | 7 |
| Epic | 9 |
| Legendary | 12 |
| Mythic | 15 |
| Godly | 19 |
| Handcrafted | 20 (the 15 hand-curated legacy NFTs; cannot be produced by merging) |

Weight = `shares × (rarityMultiplier + level + lockBoost)`, so a locked Godly NFT at the same stake carries several times the boost weight of an unlocked Plentiful one.

<figure><img src="../../../.gitbook/assets/BaseMultiplierXDC (2).jpg" alt=""><figcaption></figcaption></figure>

Higher rarity = higher `rarityMultiplier` = higher weight = larger slice of every `notifyBoost`.

{% hint style="warning" %}
`rarityMultiplier` values are **immutable** on the V3 vault. Changing them would invalidate `totalWeight` for every staked NFT, so the vault has no setter. A future change would require deploying a new vault and migrating.
{% endhint %}

---

## Merge System

Combine two NFTs of the **same rarity** to mint one NFT of the **next rarity tier**.

- Both original NFTs are burned (their psXDC shares are released back to you so you can re-stake into the new NFT, depending on the merge mode).
- A new, higher-rarity NFT is minted via `XdcStakedNFT.mintMerged` (token IDs ≥ `10000`).
- **Godly** is the highest rarity achievable through merging.

The merge system makes the collection **deflationary by design**: every merge permanently reduces the total supply. Over time, remaining NFTs become increasingly scarce and carry higher multipliers.

---

## Handcrafted NFTs

The collection includes 15 exclusive, handcrafted NFTs by the Art Director. Owners receive three additional XDC Staking NFTs. These NFTs have the **highest `rarityMultiplier`** in the collection.

---

## V3 contract stack

| Contract | Address | Role |
| --- | --- | --- |
| `XdcStakedNFT` | [`0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E`](https://xdcscan.com/address/0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E) | Fresh ERC-721 collection. Non-upgradeable. |
| `XdcNftStakingVault` (proxy) | [`0x9f38dF64eeC71e2408B24217b8D621c6B07E4Da8`](https://xdcscan.com/address/0x9f38dF64eeC71e2408B24217b8D621c6B07E4Da8) | Staking engine that holds psXDC shares under each NFT and runs the accumulator. TransparentUpgradeableProxy, ERC-7201 namespaced storage. |
| `XdcNftMigrator` | [`0x87Abbf807Be90E3c618Db410c6BEd7c1aA38556A`](https://xdcscan.com/address/0x87Abbf807Be90E3c618Db410c6BEd7c1aA38556A) | Current holder of `MIGRATOR_ROLE` (since 9 Aug 2026). The V2 → V3 migration is effectively complete — 1,431 NFTs were ported between 15 May and 4 Jul 2026 through the earlier migrators (`0x45e2…7dFb`, `0x36Fe…f026`, `0x69DE…2ea8`, all revoked). Non-upgradeable. |
| `XdcNftBoostHarvester` | [`0x6a319528111E5e50712Fd2D3d2db8323b119821D`](https://xdcscan.com/address/0x6a319528111E5e50712Fd2D3d2db8323b119821D) | Optional boost pipe: forwards native XDC into the accumulator via `feed` → `notifyBoost`. Since 14 Sep 2026 the operations wallet also holds `FEE_ROUTER_ROLE` and funds the boost directly (`notifyBoost` / `notifyBoostShares`). Non-upgradeable. |
| `LegacyMigratorBypassFacet` | [`0x2786D8Df1C38c9D4eD642B84c073349b0f0B5e13`](https://xdcscan.com/address/0x2786D8Df1C38c9D4eD642B84c073349b0f0B5e13) | Diamond facet on the legacy diamond enabling locked-NFT migration. Clears `tokenLocked`; the diamond pays the psXDC (no v2-staker call). |

→ [Staking Mechanics (V3)](xdc-staking-nfts-mechanics.md) → [Reward Model: Base NAV + Boost](xdc-nft-staking-reward-system.md) → [Migrate XDC NFTs to V3](migrate-nfts-v2-to-v3.md) → [Locked NFTs & Legacy Diamond Bypass](locked-nft-migration.md) → [Boost Harvester (technical)](boost-harvester.md) → [Smart Contract Reference (V3)](smart-contract-functions.md)
