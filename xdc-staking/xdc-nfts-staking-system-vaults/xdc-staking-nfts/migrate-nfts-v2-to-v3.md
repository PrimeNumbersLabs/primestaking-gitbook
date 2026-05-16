# Migrate XDC NFTs to V3

The V3 XDC NFT stack is a fresh set of contracts. Your legacy V2 NFTs continue to work, but to earn under the new reward model (psXDC v3 NAV + Synthetix boost slice) you migrate them through [`XdcNftMigrator`](../contract-addresses.md). Migration preserves your **tokenId**, **rarity**, and any active **lock expiry**.

{% hint style="info" %}
Migration is **one-shot per NFT**, **atomic** (all-or-nothing), and never holds your funds across transactions. Same `tokenId` end-to-end — your social presence, links, and OpenSea / PrimePort references keep working.
{% endhint %}

---

## What the migrator does

```
User                  XdcNftMigrator           Old Façade          Bridge / psXDC v3        New Vault + NFT
 │                          │                      │                      │                          │
 │ approve(migrator, id) ──►│                      │                      │                          │
 │ migrate(id, minOut)   ──►│                      │                      │                          │
 │                          │ ownerOf, getNFTData ►│                      │                          │
 │                          │ transferFrom(user→me)│                      │                          │
 │                          │ burnAndRedeem(id) ──►│                      │                          │
 │                          │ ◄────────────────── receives psXDC v2 OR native XDC                    │
 │                          │                                                                        │
 │                          │ bridge.migrate(amount) OR depositNative{value} ───────────────────────►│
 │                          │ ◄──────────────────────────────────── receives v3 shares ──────────────│
 │                          │ approve(vault, shares)                                                  │
 │                          │ mintAndStake(user, tokenId, rarity, shares) ─────────────────────────►│
 │                          │                                                                        │
 │ ◄──── new tokenId on XdcStakedNFT, staked under XdcNftStakingVault ─────────────────────────────┘
```

For locked NFTs the flow is identical except it also routes through the [`LegacyMigratorBypassFacet`](locked-nft-migration.md) to clear `tokenLocked` and preserve the original `lockEnd`.

### Properties that always hold

- **Same `tokenId`** end-to-end. Wallets, social, marketplaces keep working.
- **Same rarity** — the migrator reads `getNFTData` on the legacy NFT and mints with the matching `rarityMultiplier`.
- **Atomic** — every step reverts together. If any leg of the migration fails (slippage exceeded, bridge inactive, locked NFT but bypass facet missing, etc.) the whole transaction reverts and you keep your legacy NFT untouched.
- **No reward forfeiture** — for locked NFTs, the migrator calls `try oldFacade.claim(tokenId)` before burning so any V2-pending rewards are folded into the staked balance and survive into V3. (Vanilla `burnAndRedeem` on V2 would otherwise silently drop them.)
- **No custodial risk** — the migrator never holds funds across transactions. Any rounding dust is recoverable only by the dedicated `RESCUER_ROLE` (multisig).

---

## Steps

### 1. Open the migrate page

Go to [primestaking.xyz/xdc-nfts/migrate](https://primestaking.xyz/xdc-nfts/migrate). The page shows your **legacy** XDC NFT balance and your existing **V3** NFT balance side by side. A yellow banner also appears on `/xdc-nfts/my-nfts` if your wallet still holds legacy NFTs.

### 2. Approve the migrator

For each legacy NFT you want to migrate, approve [`XdcNftMigrator`](../contract-addresses.md) to pull it. The migrate page issues one `approve(migrator, tokenId)` per NFT.

### 3. Set slippage

The migrator passes a `minSharesOut` value through to the underlying [`PrimeStakedXDC_V3MigrationBridge`](../xdc-liquid-staking/staking-guide/migration.md) so the migration reverts if the V3 share rate moves unfavourably during execution. The default is **50 bps** (0.5%) — adjust it in the page header if conditions warrant.

### 4. Migrate

Click **Migrate** to send a single NFT, or **Migrate All** to batch through `migrateBatch(tokenIds[], minSharesOuts[])`. Confirm the transaction in your wallet.

After the transaction lands:

- Your **legacy** NFT count drops by the number of NFTs you migrated.
- Your **V3** NFT count increases by the same number — same `tokenId`s, same rarities.
- The legacy NFTs are **burned** on the old façade — there is no rollback.
- The V3 NFTs are immediately staked in [`XdcNftStakingVault`](../contract-addresses.md) and earning both NAV (from the underlying psXDC v3 shares) and the boost slice (from the Synthetix accumulator).

The migrate banner disappears from `/xdc-nfts/my-nfts` once your legacy balance reaches zero.

---

## Locked legacy NFTs

If you have a locked legacy NFT, the migrator handles it in the **same** call. Behind the scenes it talks to the [`LegacyMigratorBypassFacet`](locked-nft-migration.md) (added to the legacy Diamond via `diamondCut`) to clear the `tokenLocked` flag, then performs the standard burn + bridge + mint + stake flow. The original `lockEnd` is written to the new NFT so you can't dodge the lock by routing through migration.

If the migrator was deployed before the bypass facet was cut in, locked migrations revert with `LegacyDiamondRequiredForLockedNft(tokenId)`. The live deployment has the facet cut in — unlocked migrations are completely unaffected either way.

---

## What you see in the app afterwards

| Surface | What changes |
| --- | --- |
| `/xdc-nfts/my-nfts` | The migrated NFT appears in the V3 list with its original `tokenId`. |
| `/xdc-nfts/[tokenId]` | The detail page reads from the V3 vault — `stakedShares`, `weight`, pending boost, lock status. |
| Migrate page | Migrated NFTs appear under "Your v3 NFTs" with a **Migrated** badge for traceability. |
| Activity Log | A `Migrated(user, tokenId, rarity, …)` entry per NFT. |

You can interact with your V3 NFT immediately: stake more shares, lock, merge, claim boost, or `burnAndRedeem`.

---

## FAQ

**Can I migrate part of an NFT?** No. Migration is one tokenId at a time. You can batch multiple NFTs in a single tx via `migrateBatch`.

**Can I roll back?** No. Legacy NFTs are **burned** when migrated. The V3 NFT is functionally equivalent (same `tokenId`, same rarity, same lock expiry).

**Do I need to claim V2 rewards first?** No. The migrator handles the V2-side claim automatically (best-effort `try claim(tokenId)`) so pending V2 rewards are folded into the staked balance.

**What happens to my V2 psXDC staked inside the NFT?** It's redeemed from the old façade, bridged into V3 shares through the [`PrimeStakedXDC_V3MigrationBridge`](../xdc-liquid-staking/staking-guide/migration.md), and staked into the new vault under your new NFT — same `tokenId`, same rarity.

→ [Locked NFTs & Legacy Diamond Bypass](locked-nft-migration.md) → [Boost Harvester (technical)](boost-harvester.md) → [Smart Contract Reference (V3)](smart-contract-functions.md) → [Deployed Contracts & Addresses](../contract-addresses.md)
