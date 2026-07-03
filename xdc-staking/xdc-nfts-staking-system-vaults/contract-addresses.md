---
description: Canonical address book for PrimeStaking V3.1, covering the psXDC vault, migration bridge, airdrop distributor, XDC NFT v3 stack, and legacy contracts.
---

# Deployed Contracts & Addresses

{% hint style="info" %}
All V3.1 contracts are live on **XDC Mainnet (chain ID `50`)**. In July 2026 the protocol redeployed the liquid-staking vault as **`PrimeStakedXDC_V3_1`** and mirrored every V3 holder's balance into it 1:1 via a snapshot airdrop; no user action was required. See [V2 vs V3: What Changed](xdc-liquid-staking/v2-vs-v3.md).
{% endhint %}

This page is the single source of truth for every address the V3.1 stack interacts with. The UI, indexer, and partner integrations are all wired against the addresses listed here.

---

## V3.1: XDC Liquid Staking (psXDC)

| Contract | Address | Type | Notes |
| --- | --- | --- | --- |
| **`PrimeStakedXDC_V3_1`** | [`0xa7FD1c5601348633018003C90aE568d1ff7973e4`](https://xdcscan.com/address/0xa7FD1c5601348633018003C90aE568d1ff7973e4) | ERC-4626 vault (native XDC) | **The live psXDC token and vault.** Non-upgradeable, deployed with a regular constructor, no proxy. All staking, withdrawals, and share-price reads go here. |
| **`PrimeStakedXDC_V3MigrationBridge`** (v2 → V3.1) | [`0x6c57075c7A157113D369109B78738A798d41373C`](https://xdcscan.com/address/0x6c57075c7A157113D369109B78738A798d41373C) | Migration bridge | Burns V2 psXDC and mints V3.1 shares via `migrate(amount, minSharesOut)`. This is the bridge for v2 stragglers who had not migrated before the snapshot. |
| **`V31AirdropDistributor`** | [`0x3b7185844451a57CC45807243477e442ff1A3553`](https://xdcscan.com/address/0x3b7185844451a57CC45807243477e442ff1A3553) | Airdrop distributor (**finalized**) | Batch-minted the snapshot balances of every V3 holder onto V3.1 at launch. Permanently finalized; it can never mint again. Kept for on-chain audit trail. |

## Superseded: old V3 (do not use)

| Contract | Address | Status |
| --- | --- | --- |
| `PrimeStakedXDC_V3` (old vault) | [`0x98D916F5773Ac0482b49856f2659d6c32114C4Ba`](https://xdcscan.com/address/0x98D916F5773Ac0482b49856f2659d6c32114C4Ba) | **Superseded.** Every holder balance at the snapshot was mirrored 1:1 onto V3.1. Old V3 tokens have no further function; its bridge has been cut to a dead address. |
| Old v2 → V3 bridge | [`0x2927630dfDd66433DbA9370b316EF5a8408d5dD2`](https://xdcscan.com/address/0x2927630dfDd66433DbA9370b316EF5a8408d5dD2) | **Superseded** by the v2 → V3.1 bridge above. |

---

## V3: XDC NFTs

The NFT vault proxy was upgraded in place during the V3.1 cutover, so the user-facing addresses (collection + vault proxy) are unchanged. The migrator, harvester, and bypass facet were redeployed against V3.1.

| Contract | Address | Type | Notes |
| --- | --- | --- | --- |
| **`XdcStakedNFT`** | [`0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E`](https://xdcscan.com/address/0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E) | ERC-721 collection | **Non-upgradeable**. Stores rarity on-chain; migrated NFTs preserve their legacy `tokenId` (except legacy ids ≥ `10000`, which the migrator remaps into the `5558–9999` band). |
| **`XdcNftStakingVault`** (proxy) | [`0x9f38dF64eeC71e2408B24217b8D621c6B07E4Da8`](https://xdcscan.com/address/0x9f38dF64eeC71e2408B24217b8D621c6B07E4Da8) | TransparentUpgradeableProxy | User-facing vault for stake / withdraw / claim / lock / merge / `burnAndRedeem`. Uses ERC-7201 namespaced storage. Now holds psXDC **V3.1** shares. |
| `XdcNftStakingVault` implementation | [`0x697175A252E4F832F2D43A6E5933ADb75d9Eef1E`](https://xdcscan.com/address/0x697175A252E4F832F2D43A6E5933ADb75d9Eef1E) | Vault logic | Upgraded July 2026 to add `migrateV3Token` (V3 → V3.1 cutover). Always interact with the proxy above, not this address. |
| Vault `ProxyAdmin` | [`0xCE17925533C570B3EE5621Df39019b1a5785fb3d`](https://xdcscan.com/address/0xCE17925533C570B3EE5621Df39019b1a5785fb3d) | Proxy admin | Owned by the protocol multisig. |
| **`XdcNftMigratorV2`** (V3.1 wiring) | [`0x69DE30161ec0f2e0Dc0649190dB9b93F4c492ea8`](https://xdcscan.com/address/0x69DE30161ec0f2e0Dc0649190dB9b93F4c492ea8) | Migrator (**live**) | The active one-shot migrator: burns legacy NFT → bridges psXDC through the v2 → V3.1 bridge → mints + stakes new NFT. Remaps legacy ids ≥ `10000` into `5558–9999`, emitting `LegacyIdRemapped`. **Non-upgradeable**. |
| **`XdcNftBoostHarvester`** (V3.1 wiring) | [`0x6a319528111E5e50712Fd2D3d2db8323b119821D`](https://xdcscan.com/address/0x6a319528111E5e50712Fd2D3d2db8323b119821D) | Boost pipe | Funds the Synthetix-style boost accumulator via `notifyBoost`. **Non-upgradeable**. |
| **`LegacyMigratorBypassFacet`** (V3.1 wiring) | [`0x2786D8Df1C38c9D4eD642B84c073349b0f0B5e13`](https://xdcscan.com/address/0x2786D8Df1C38c9D4eD642B84c073349b0f0B5e13) | Diamond facet (**live**) | Added to the legacy Diamond via `diamondCut` so locked legacy NFTs migrate in a single transaction. Clears the diamond's `tokenLocked` flag and lets the diamond's own `burnAndRedeem` pay the psXDC. |

### Superseded NFT-stack contracts (roles revoked, do not use)

| Contract | Address | Status |
| --- | --- | --- |
| Old `XdcNftMigratorV2` (V3 wiring) | [`0x36Fe37Ca1FEF0e409977a1c28d191B55333cf026`](https://xdcscan.com/address/0x36Fe37Ca1FEF0e409977a1c28d191B55333cf026) | `MINTER_ROLE` / `MIGRATOR_ROLE` revoked at the V3.1 cutover. |
| Old `XdcNftBoostHarvester` | [`0x3bEdb37FC873F64BEeFCA551b3A836e59fc18DeA`](https://xdcscan.com/address/0x3bEdb37FC873F64BEeFCA551b3A836e59fc18DeA) | `FEE_ROUTER_ROLE` revoked at the V3.1 cutover. |
| Old `LegacyMigratorBypassFacet` | [`0x64413bAD206b5D90a5010cc683F50086407F25C6`](https://xdcscan.com/address/0x64413bAD206b5D90a5010cc683F50086407F25C6) | Replaced on the legacy Diamond via `diamondCut`. |
| Original `XdcNftMigrator` | [`0x45e2e91098A8451EA450754784e043bb3F8C7dFb`](https://xdcscan.com/address/0x45e2e91098A8451EA450754784e043bb3F8C7dFb) | Paused and replaced in June 2026. |

---

## Spot Trading (psXDC/XDC DEX)

The protocol-owned spot venue behind [primestaking.xyz/spot](https://primestaking.xyz/spot). The factory and router are token-agnostic and carried over from the original deployment; the pair and limit-order book were redeployed against V3.1 in July 2026.

| Contract | Address | Type | Notes |
| --- | --- | --- | --- |
| **psXDC/WXDC pair (V3.1)** | [`0x14A11af8980ea7e18B18cAbf4721B61586Bab087`](https://xdcscan.com/address/0x14A11af8980ea7e18B18cAbf4721B61586Bab087) | UniswapV2Pair | The live trading pool, seeded at NAV. Holds V3.1 psXDC and canonical WXDC. |
| **`SpotLimitOrders`** (V3.1) | [`0x89dB7715fFc5B8b2C4A604BdD49b006df201247a`](https://xdcscan.com/address/0x89dB7715fFc5B8b2C4A604BdD49b006df201247a) | Limit-order book | Non-custodial on-chain limit orders settled against the pair. Makers can always cancel for a full refund of unfilled escrow. |
| `UniswapV2Router02` | [`0xf77440C4Dc3Dcd5Bb93DaA863BF93Fc306EC0791`](https://xdcscan.com/address/0xf77440C4Dc3Dcd5Bb93DaA863BF93Fc306EC0791) | Router | Swap and liquidity entry point; wired to the canonical WXDC. |
| `UniswapV2Factory` | [`0x3a718EB1b4b06968F78a0a3b7e3dF07037E83f5d`](https://xdcscan.com/address/0x3a718EB1b4b06968F78a0a3b7e3dF07037E83f5d) | Factory | PrimeStaking-owned; created both the old and new pairs. |
| `WXDC` (canonical) | [`0x951857744785E80e2De051c32EE7b25f9c458C42`](https://xdcscan.com/address/0x951857744785E80e2De051c32EE7b25f9c458C42) | Wrapped XDC | Shared network-wide wrapper, not operated by PrimeStaking. |

### Superseded spot contracts (do not use)

| Contract | Address | Status |
| --- | --- | --- |
| Old psXDC/WXDC pair | [`0x43a4557d0AF08929680387B2A6edae8068C02FE6`](https://xdcscan.com/address/0x43a4557d0AF08929680387B2A6edae8068C02FE6) | Holds the retired old-V3 token. LPs were credited their psXDC side in the snapshot airdrop; the WXDC side remains withdrawable by removing liquidity. |
| Old `SpotLimitOrders` | [`0x6C398e05BcDbc34b7A92befA8D6e6bdFA28322Fb`](https://xdcscan.com/address/0x6C398e05BcDbc34b7A92befA8D6e6bdFA28322Fb) | **Paused** July 2026. Fills are blocked; cancels stay enabled and expired orders are being swept automatically, refunding makers. |

---

## Partner Staking

These power the white-label [Partner Staking](../../partner-staking/README.md) product. The registry is shared; each partner pool is its own independent `PartnerStakedXDC_V3` deployment, so pool addresses are discovered dynamically via the registry rather than hard-coded here.

| Contract | Address | Type | Notes |
| --- | --- | --- | --- |
| **`PartnerVaultRegistry`** | [`0x325DEEA5C7c0Ce0D774c4A67EcCaAf1cF8953a67`](https://xdcscan.com/address/0x325DEEA5C7c0Ce0D774c4A67EcCaAf1cF8953a67) | Registry (`Ownable2Step`) | On-chain directory of partner pools. Codehash-gated registration; PrimeStaking curates which pools are **Verified**. Enumerate pools via `allVaults()` / `verifiedVaults()`. |
| `PartnerStakedXDC_V3` (per partner) | _discover via registry_ | ERC-4626 vault (native XDC) | Each partner deploys their own pool. Identical runtime bytecode (15% fee constant) so the registry can verify it by `codehash`. |

---

## Legacy (V2)

| Contract | Address | Role |
| --- | --- | --- |
| Legacy Diamond (ERC-2535) | [`0x7a5d364b97126600C0AdDFD5C339230748bcaA17`](https://xdcscan.com/address/0x7a5d364b97126600C0AdDFD5C339230748bcaA17) | Host of all legacy XDC NFT facets. Also hosts the live `LegacyMigratorBypassFacet` (`0x2786…5e13`) for one-tx migration. |
| Drained v2 staker (`PrimeStakerV2XDC`) | [`0x2204E4Db4D45A290e284daa6f6fb52273593B293`](https://xdcscan.com/address/0x2204E4Db4D45A290e284daa6f6fb52273593B293) | The original v2 XDC staker. Drained to ~0; retained for reference only. |
| Legacy ERC-721 façade | [`0x9D458330e458f11fd1cE7E44B3a66568af8076a0`](https://xdcscan.com/address/0x9D458330e458f11fd1cE7E44B3a66568af8076a0) | The user-visible NFT contract behind the V2 XDC NFTs. |
| Legacy psXDC v2 (proxy) | [`0x9B8e12b0BAC165B86967E771d98B520Ec3F665A6`](https://xdcscan.com/address/0x9B8e12b0BAC165B86967E771d98B520Ec3F665A6) | The V2 psXDC ERC-20 token. Use this address when approving the v2 → V3.1 migration bridge. |

---

## Quick reference for integrators

If you are integrating PrimeStaking from a partner application, the only contracts you ever call directly are:

- `PrimeStakedXDC_V3_1` (`0xa7FD…73e4`) to stake, withdraw, redeem, or read the share price.
- `PrimeStakedXDC_V3MigrationBridge` (`0x6c57…373C`), only if your users hold V2 psXDC and want to migrate.
- `XdcNftStakingVault` (proxy, `0x9f38…4da8`) for any NFT-side action.
- `XdcNftMigratorV2` (`0x69DE…2ea8`), only for migrating legacy V2 NFTs.

The harvester, bypass facet, distributor, vault implementation, and ProxyAdmin are operated by the protocol and do not need to be called from partner code.

→ [V3 Architecture](xdc-liquid-staking/v3-architecture.md) → [Smart Contract Reference (psXDC v3)](xdc-liquid-staking/smart-contract-functions.md) → [Smart Contract Reference (XDC NFT v3)](xdc-staking-nfts/smart-contract-functions.md)
