---
description: Canonical address book for PrimeStaking V3 — psXDC vault, migration bridge, XDC NFT v3 stack, and legacy contracts kept available for users who have not yet migrated.
---

# Deployed Contracts & Addresses

{% hint style="info" %}
All V3 contracts are live on **XDC Mainnet (chain ID `50`)**. The legacy V2 contracts continue to operate for users who have not migrated — see [Legacy (V2 — historical)](../../legacy/README.md).
{% endhint %}

This page is the single source of truth for every address the V3 stack interacts with. The UI, indexer, and partner integrations are all wired against the addresses listed here.

---

## V3 — XDC Liquid Staking (psXDC)

| Contract | Address | Type | Notes |
| --- | --- | --- | --- |
| **`PrimeStakedXDC_V3`** | [`0x98D916F5773Ac0482b49856f2659d6c32114C4Ba`](https://xdcscan.com/address/0x98D916F5773Ac0482b49856f2659d6c32114C4Ba) | ERC-4626 vault (native XDC) | psXDC v3 share token. **Non-upgradeable** — deployed with a regular constructor, no proxy. |
| **`PrimeStakedXDC_V3MigrationBridge`** | [`0x2927630dfDd66433DbA9370b316EF5a8408d5dD2`](https://xdcscan.com/address/0x2927630dfDd66433DbA9370b316EF5a8408d5dD2) | Migration bridge | Burns V2 psXDC and mints V3 shares via `migrate(amount, minSharesOut)`. Time-locked treasury controls, daily withdrawal cap. |

---

## V3 — XDC NFTs

| Contract | Address | Type | Notes |
| --- | --- | --- | --- |
| **`XdcStakedNFT`** | [`0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E`](https://xdcscan.com/address/0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E) | ERC-721 collection | **Non-upgradeable**. Stores rarity on-chain; migrated NFTs preserve their legacy `tokenId`. |
| **`XdcNftStakingVault`** (proxy) | [`0x9f38dF64eeC71e2408B24217b8D621c6B07E4Da8`](https://xdcscan.com/address/0x9f38dF64eeC71e2408B24217b8D621c6B07E4Da8) | TransparentUpgradeableProxy | User-facing vault for stake / withdraw / claim / lock / merge / `burnAndRedeem`. Uses ERC-7201 namespaced storage. |
| `XdcNftStakingVault` implementation | [`0x98D4880663ABAF96CAb2C005eb4A56bf1d583990`](https://xdcscan.com/address/0x98D4880663ABAF96CAb2C005eb4A56bf1d583990) | Vault logic | Always interact with the proxy above, not this address. |
| Vault `ProxyAdmin` | [`0xCE17925533C570B3EE5621Df39019b1a5785fb3d`](https://xdcscan.com/address/0xCE17925533C570B3EE5621Df39019b1a5785fb3d) | Proxy admin | Owned by the protocol multisig. |
| **`XdcNftMigrator`** | [`0x45e2e91098A8451EA450754784e043bb3F8C7dFb`](https://xdcscan.com/address/0x45e2e91098A8451EA450754784e043bb3F8C7dFb) | Migrator | One-shot per NFT: burns legacy NFT → bridges psXDC → mints + stakes new NFT. **Non-upgradeable**. |
| **`XdcNftBoostHarvester`** | [`0x3bEdb37FC873F64BEeFCA551b3A836e59fc18DeA`](https://xdcscan.com/address/0x3bEdb37FC873F64BEeFCA551b3A836e59fc18DeA) | Boost pipe | Funds the Synthetix-style boost accumulator via `notifyBoost`. **Non-upgradeable**. |
| **`LegacyMigratorBypassFacet`** | [`0x275641d5bA81786A7e60352F990F0c203e7D1836`](https://xdcscan.com/address/0x275641d5bA81786A7e60352F990F0c203e7D1836) | Diamond facet | Added to the legacy Diamond via `diamondCut` so locked legacy NFTs can be migrated in a single transaction. |

---

## Legacy (V2) — still operational

These contracts are kept live so users who choose not to migrate can continue to use the V2 product. Every new deposit and every audited V3 path uses the V3 contracts above.

| Contract | Address | Role |
| --- | --- | --- |
| Legacy Diamond (ERC-2535) | [`0x7a5d364b97126600C0AdDFD5C339230748bcaA17`](https://xdcscan.com/address/0x7a5d364b97126600C0AdDFD5C339230748bcaA17) | Host of all legacy XDC NFT facets. Now also hosts `LegacyMigratorBypassFacet` for one-tx migration. |
| Legacy ERC-721 façade | [`0x9D458330e458f11fd1cE7E44B3a66568af8076a0`](https://xdcscan.com/address/0x9D458330e458f11fd1cE7E44B3a66568af8076a0) | The user-visible NFT contract behind the V2 XDC NFTs. |
| Legacy psXDC v2 (proxy) | [`0x9B8e12b0BAC165B86967E771d98B520Ec3F665A6`](https://xdcscan.com/address/0x9B8e12b0BAC165B86967E771d98B520Ec3F665A6) | The V2 psXDC ERC-20 token. Use this address when approving the V2→V3 migration bridge. |

---

## PRFI (Base Mainnet, chain ID `8453`)

PRFI is a separate product and is **not** affected by the V3 rebuild.

| Contract | Address | Role |
| --- | --- | --- |
| PRFI NFT collection | [`0x693A3A45Ff596024f844Be1cc6845d59F778dCF5`](https://basescan.org/address/0x693A3A45Ff596024f844Be1cc6845d59F778dCF5) | PRFI staking NFTs on Base. |
| PRFI ERC-20 | [`0x7BBCf1B600565AE023a1806ef637Af4739dE3255`](https://basescan.org/address/0x7BBCf1B600565AE023a1806ef637Af4739dE3255) | PRFI token on Base. |

---

## Quick reference for integrators

If you are integrating PrimeStaking from a partner application, the only contracts you ever call directly are:

- `PrimeStakedXDC_V3` — to stake, withdraw, redeem, or read the share price.
- `PrimeStakedXDC_V3MigrationBridge` — only if your users hold V2 psXDC and want to migrate.
- `XdcNftStakingVault` (proxy) — for any NFT-side action.
- `XdcNftMigrator` — only during the NFT migration window.

The harvester, bypass facet, vault implementation, and ProxyAdmin are operated by the protocol and do not need to be called from partner code.

→ [V3 Architecture](xdc-liquid-staking/v3-architecture.md) → [Smart Contract Reference (psXDC v3)](xdc-liquid-staking/smart-contract-functions.md) → [Smart Contract Reference (XDC NFT v3)](xdc-staking-nfts/smart-contract-functions.md)
