# XDC NFTs

XDC Staking NFTs are gamified staking positions on the XDC Network. Each NFT holds **psXDC v3 vault shares** and earns two stacked yields: a **base ~4.5%** from the underlying share-price appreciation (always earned, no claim, no rarity dependency) and an additional **boost slice up to ~1.5%** from a Synthetix-style accumulator funded by the protocol's reward harvester, weighted by rarity, level, and lock status.

{% hint style="info" %}
The XDC NFT stack has been rebuilt around the psXDC v3 vault. New contract addresses: [`XdcStakedNFT`](../contract-addresses.md), [`XdcNftStakingVault`](../contract-addresses.md), [`XdcNftMigrator`](../contract-addresses.md), [`XdcNftBoostHarvester`](../contract-addresses.md), [`LegacyMigratorBypassFacet`](../contract-addresses.md). The legacy V2 collection at `0x9D45…76a0` remains operational for any holder who has not yet migrated — see [Migrate XDC NFTs to V3](migrate-nfts-v2-to-v3.md).
{% endhint %}

---

## Key Facts

| | |
| --- | --- |
| **Collection size** | 5,557 NFTs (5,542 generative + 15 handcrafted), preserved across V2 → V3 |
| **Token staked** | **psXDC v3 vault shares** (not raw XDC) |
| **Base yield** | ~4.5% — share-price appreciation of the underlying psXDC |
| **Boost slice** | Up to ~1.5% — Synthetix accumulator, weighted by rarity / level / lock |
| **Floor APY** | **~4.5%** — the base NAV, automatic, always earned regardless of rarity / lock / boost cadence |
| **Target APY band (with boost)** | ~4.75% (unlocked) → ~6% (locked) when boost stream is steady |
| **Reward token (boost)** | XDC — `notifyBoost` mints shares, claim unwraps to native XDC |
| **Locked yield** | Additive `lockBoost` term added to NFT weight when locked |
| **Merge** | Two same-rarity NFTs → one higher-rarity NFT (originals burned) |
| **Marketplace** | [PrimePort.xyz](https://primeport.xyz) |

---

## How It Works

1. **Get psXDC shares** — stake XDC in [`PrimeStakedXDC_V3`](../xdc-liquid-staking/README.md) or buy psXDC on a DEX. (Already hold V2 psXDC? [Migrate to V3 first](../xdc-liquid-staking/staking-guide/migration.md).)
2. **Get an NFT** — buy one on [PrimePort](https://primeport.xyz), or migrate a legacy V2 NFT through [`XdcNftMigrator`](migrate-nfts-v2-to-v3.md) (preserves your `tokenId`, rarity, and any active lock).
3. **Stake psXDC shares into your NFT** — the vault records the shares against the NFT's `tokenId`. The NFT's **weight** in the boost accumulator becomes `stakedShares × (rarityMultiplier + level + lockBoost)`.
4. **Earn two stacked yields**:
   - **Base NAV** — your staked shares keep appreciating; you receive them back at the higher value when you withdraw.
   - **Boost** — every `notifyBoost` push from the harvester increments `rewardPerWeightStored`; your earned slice grows in proportion to your weight.
5. **Claim boost** from the NFT detail page whenever you want — it is paid in XDC. Base NAV is automatic and needs no claim.
6. **Upgrade** — merge two same-rarity NFTs into a higher-tier one for a larger `rarityMultiplier`.
7. **Lock (optional)** — locking adds `lockBoost` to the weight calculation. Lock expiry is preserved across migration so users can't dodge the lock by routing through the migrator.
8. **`burnAndRedeem`** — burns the NFT and returns the underlying psXDC shares (or, optionally, redeems them to XDC in one transaction).

---

## Rarity Tiers

Each NFT has a rarity that determines its `rarityMultiplier`, which feeds into the weight formula:

| Rarity | Base Multiplier |
| --- | --- |
| Plentiful | 0.3 |
| Common | 0.4 |
| Uncommon | 0.5 |
| Rare | 0.7 |
| Epic | 0.9 |
| Legendary | 1.2 |
| Mythic | 1.5 |
| Godly | 1.9 |

<figure><img src="../../../.gitbook/assets/BaseMultiplierXDC (2).jpg" alt=""><figcaption></figcaption></figure>

Higher rarity = higher `rarityMultiplier` = higher weight = larger slice of every `notifyBoost`.

{% hint style="warning" %}
`rarityMultiplier` values are **immutable** on the V3 vault. Changing them would invalidate `totalWeight` for every staked NFT, so the vault has no setter — a future change would require deploying a new vault and migrating.
{% endhint %}

---

## Merge System

Combine two NFTs of the **same rarity** to mint one NFT of the **next rarity tier**.

- Both original NFTs are burned (their psXDC shares are released back to you so you can re-stake into the new NFT, depending on the merge mode).
- A new, higher-rarity NFT is minted via `XdcStakedNFT.mintMerged` (token IDs ≥ `10000`).
- **Godly** is the highest rarity achievable through merging.

The merge system makes the collection **deflationary by design** — every merge permanently reduces the total supply. Over time, remaining NFTs become increasingly scarce and carry higher multipliers.

---

## Handcrafted NFTs

The collection includes 15 exclusive, handcrafted NFTs by the Art Director. Owners receive three additional XDC Staking NFTs. These NFTs have the **highest `rarityMultiplier`** in the collection.

---

## V3 contract stack

| Contract | Address | Role |
| --- | --- | --- |
| `XdcStakedNFT` | [`0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E`](https://xdcscan.com/address/0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E) | Fresh ERC-721 collection. Non-upgradeable. |
| `XdcNftStakingVault` (proxy) | [`0x9f38dF64eeC71e2408B24217b8D621c6B07E4Da8`](https://xdcscan.com/address/0x9f38dF64eeC71e2408B24217b8D621c6B07E4Da8) | Staking engine — holds psXDC shares under each NFT, runs the accumulator. TransparentUpgradeableProxy, ERC-7201 namespaced storage. |
| `XdcNftMigrator` | [`0x45e2e91098A8451EA450754784e043bb3F8C7dFb`](https://xdcscan.com/address/0x45e2e91098A8451EA450754784e043bb3F8C7dFb) | One-shot V2 → V3 migrator. Non-upgradeable. |
| `XdcNftBoostHarvester` | [`0x3bEdb37FC873F64BEeFCA551b3A836e59fc18DeA`](https://xdcscan.com/address/0x3bEdb37FC873F64BEeFCA551b3A836e59fc18DeA) | Funds the boost accumulator via `notifyBoost`. Non-upgradeable. |
| `LegacyMigratorBypassFacet` | [`0x275641d5bA81786A7e60352F990F0c203e7D1836`](https://xdcscan.com/address/0x275641d5bA81786A7e60352F990F0c203e7D1836) | Diamond facet on the legacy diamond enabling locked-NFT migration. |

→ [Staking Mechanics (V3)](xdc-staking-nfts-mechanics.md) → [Reward Model: Base NAV + Boost](xdc-nft-staking-reward-system.md) → [Migrate XDC NFTs to V3](migrate-nfts-v2-to-v3.md) → [Locked NFTs & Legacy Diamond Bypass](locked-nft-migration.md) → [Boost Harvester (technical)](boost-harvester.md) → [Smart Contract Reference (V3)](smart-contract-functions.md)
