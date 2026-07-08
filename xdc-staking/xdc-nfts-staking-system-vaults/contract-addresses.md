---
description: Canonical address book for PrimeStaking V3.2, covering the psXDC vault, airdrop distributor, referral program, XDC NFT v3 stack, spot venue, and legacy contracts.
---

# Deployed Contracts & Addresses

{% hint style="info" %}
All V3.2 contracts are live on **XDC Mainnet (chain ID `50`)**. In July 2026 the protocol redeployed the liquid-staking vault as **`PrimeStakedXDC_V3_2`** and mirrored every V3.1 holder's balance into it 1:1 via a snapshot airdrop; no user action was required. The XDC NFT vault was repointed to the V3.2 token in the same operation. See [V2 vs V3: What Changed](xdc-liquid-staking/v2-vs-v3.md).
{% endhint %}

This page is the single source of truth for every address the V3.2 stack interacts with. The UI, indexer, and partner integrations are all wired against the addresses listed here.

---

## V3.2: XDC Liquid Staking (psXDC)

| Contract | Address | Type | Notes |
| --- | --- | --- | --- |
| **`PrimeStakedXDC_V3_2`** | [`0xDc74c0DaED82ae94486DeeF22991d2F54173c734`](https://xdcscan.com/address/0xDc74c0DaED82ae94486DeeF22991d2F54173c734) | ERC-4626 vault (native XDC) | **The live psXDC token and vault.** Non-upgradeable, deployed with a regular constructor, no proxy. All staking, withdrawals, and share-price reads go here. Permanent-ledger model: rewards accrue at any backing level via the manager reward lane. |
| **`V32AirdropDistributor`** | [`0x9ADb9Dfc340375113042b4722e780D34B1123e47`](https://xdcscan.com/address/0x9ADb9Dfc340375113042b4722e780D34B1123e47) | Airdrop distributor (**finalized**) | Batch-minted the snapshot balances of every V3.1 holder onto V3.2 at launch. Permanently finalized; it can never mint again. Kept for on-chain audit trail. |
| **`PrimeStakedXDC_V3_2MigrationBridge`** (v2 → V3.2) | [`0x313e8d6Ad3D16be6318dF2AF5a54A87Aea42c280`](https://xdcscan.com/address/0x313e8d6Ad3D16be6318dF2AF5a54A87Aea42c280) | Migration bridge | For V2 stragglers only: burns V2 psXDC and mints V3.2 shares 1:1. It never accepts V3.1 (the airdrop already credited every V3.1 holder 1:1). The vault's `migrationBridge` repoint to this address is scheduled behind the 24h governance timelock; migration reopens in the app once it executes. |

## Referral program

| Contract | Address | Type | Notes |
| --- | --- | --- | --- |
| **`ReferralRegistry`** | [`0x9765BE3fd9e0d450bD46Ef2A30A0Feb4B15616B3`](https://xdcscan.com/address/0x9765BE3fd9e0d450bD46Ef2A30A0Feb4B15616B3) | Referral binder | `stakeWithReferral(referrer)` binds one immutable referrer to the caller and forwards the stake to the vault in one transaction. Minimum bind stake: 100 XDC. |
| **`ReferralRewards`** | [`0xD910E7E0dC457Ccd425E9a5cF716b0F6B7045549`](https://xdcscan.com/address/0xD910E7E0dC457Ccd425E9a5cF716b0F6B7045549) | Epoch Merkle distributor | Pays referrer fee-share per epoch against a posted Merkle root. Roots are immutable per epoch and can never be posted underfunded. |

## Superseded liquid-staking vaults (do not use)

| Contract | Address | Status |
| --- | --- | --- |
| `PrimeStakedXDC_V3_1` (prior vault) | [`0xa7FD1c5601348633018003C90aE568d1ff7973e4`](https://xdcscan.com/address/0xa7FD1c5601348633018003C90aE568d1ff7973e4) | **Superseded and paused.** Every holder balance at the snapshot was mirrored 1:1 onto V3.2. The V3.1 token is frozen and has no further function. |
| `PrimeStakedXDC_V3` (original vault) | [`0x98D916F5773Ac0482b49856f2659d6c32114C4Ba`](https://xdcscan.com/address/0x98D916F5773Ac0482b49856f2659d6c32114C4Ba) | **Superseded** (V3 → V3.1 → V3.2). No further function. |
| `V31AirdropDistributor` | [`0x3b7185844451a57CC45807243477e442ff1A3553`](https://xdcscan.com/address/0x3b7185844451a57CC45807243477e442ff1A3553) | The prior (V3 → V3.1) airdrop distributor, finalized. |

---

## V3: XDC NFTs

The NFT vault proxy was upgraded in place during the V3.2 cutover, so the user-facing addresses (collection + vault proxy) are unchanged. The vault now holds psXDC V3.2 shares and supports multiple lock tiers with expiring boosts.

| Contract | Address | Type | Notes |
| --- | --- | --- | --- |
| **`XdcStakedNFT`** | [`0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E`](https://xdcscan.com/address/0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E) | ERC-721 collection | **Non-upgradeable**. Stores rarity on-chain; migrated NFTs preserve their legacy `tokenId` (except legacy ids ≥ `10000`, which the migrator remaps into the `5558–9999` band). |
| **`XdcNftStakingVault`** (proxy) | [`0x9f38dF64eeC71e2408B24217b8D621c6B07E4Da8`](https://xdcscan.com/address/0x9f38dF64eeC71e2408B24217b8D621c6B07E4Da8) | TransparentUpgradeableProxy | User-facing vault for stake / withdraw / claim / lock / merge / `burnAndRedeem`. Uses ERC-7201 namespaced storage. Now holds psXDC **V3.2** shares. |
| `XdcNftStakingVault` implementation | [`0xF39b759c03B593C34d2905E0991F2cf45d8D148B`](https://xdcscan.com/address/0xF39b759c03B593C34d2905E0991F2cf45d8D148B) | Vault logic | Upgraded July 2026 for the V3.2 cutover: repoints psXDC via `migrateV3Token`, adds multi-tier locks (30/90/180/365 days) with **boost expiry**, and a permissionless `pokeExpired` keeper hook. Always interact with the proxy above, not this address. |
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

The protocol-owned spot venue behind [primestaking.xyz/spot](https://primestaking.xyz/spot). The factory and router are token-agnostic and carried over from the original deployment; the pair and limit-order book were redeployed against V3.2 in July 2026.

| Contract | Address | Type | Notes |
| --- | --- | --- | --- |
| **psXDC/WXDC pair (V3.2)** | [`0x60189924A43947bE7Bf0350E81be0D2f00D95034`](https://xdcscan.com/address/0x60189924A43947bE7Bf0350E81be0D2f00D95034) | UniswapV2Pair | The live trading pool, seeded at NAV (1:1). Holds V3.2 psXDC and canonical WXDC. |
| **`SpotLimitOrders`** (V3.2) | [`0xcd81d7d83101884D0B070153A0EBC9cD6C14f6B9`](https://xdcscan.com/address/0xcd81d7d83101884D0B070153A0EBC9cD6C14f6B9) | Limit-order book | Non-custodial on-chain limit orders settled against the pair. Makers can always cancel for a full refund of unfilled escrow. |
| `UniswapV2Router02` | [`0xf77440C4Dc3Dcd5Bb93DaA863BF93Fc306EC0791`](https://xdcscan.com/address/0xf77440C4Dc3Dcd5Bb93DaA863BF93Fc306EC0791) | Router | Swap and liquidity entry point; wired to the canonical WXDC. |
| `UniswapV2Factory` | [`0x3a718EB1b4b06968F78a0a3b7e3dF07037E83f5d`](https://xdcscan.com/address/0x3a718EB1b4b06968F78a0a3b7e3dF07037E83f5d) | Factory | PrimeStaking-owned; created every pair. |
| `WXDC` (canonical) | [`0x951857744785E80e2De051c32EE7b25f9c458C42`](https://xdcscan.com/address/0x951857744785E80e2De051c32EE7b25f9c458C42) | Wrapped XDC | Shared network-wide wrapper, not operated by PrimeStaking. |

### Superseded spot contracts (do not use)

| Contract | Address | Status |
| --- | --- | --- |
| psXDC/WXDC pair (V3.1) | [`0x14A11af8980ea7e18B18cAbf4721B61586Bab087`](https://xdcscan.com/address/0x14A11af8980ea7e18B18cAbf4721B61586Bab087) | Holds the now-frozen V3.1 token. LPs were credited their psXDC side in the V3.2 snapshot airdrop; because V3.1 is frozen the pool can no longer be unwound on-chain, so the remaining WXDC leg is refunded manually via the migration-exit page. |
| `SpotLimitOrders` (V3.1) | [`0x89dB7715fFc5B8b2C4A604BdD49b006df201247a`](https://xdcscan.com/address/0x89dB7715fFc5B8b2C4A604BdD49b006df201247a) | **Paused.** SELL escrow (psXDC) was migrated to V3.2 in the airdrop; BUY orders escrow native XDC and can still be cancelled for a full refund. |
| Old psXDC/WXDC pair (V3) | [`0x43a4557d0AF08929680387B2A6edae8068C02FE6`](https://xdcscan.com/address/0x43a4557d0AF08929680387B2A6edae8068C02FE6) | Retired original-V3 pool. |

---

## Partner Staking

These power the white-label [Partner Staking](../../partner-staking/README.md) product. The registry is shared; each partner pool is its own independent `PartnerStakedXDC_V3` deployment, so pool addresses are discovered dynamically via the registry rather than hard-coded here.

| Contract | Address | Type | Notes |
| --- | --- | --- | --- |
| **`PartnerVaultRegistry`** | [`0x325DEEA5C7c0Ce0D774c4A67EcCaAf1cF8953a67`](https://xdcscan.com/address/0x325DEEA5C7c0Ce0D774c4A67EcCaAf1cF8953a67) | Registry (`Ownable2Step`) | On-chain directory of partner pools. Codehash-gated registration; PrimeStaking curates which pools are **Verified**. Enumerate pools via `allVaults()` / `verifiedVaults()`. |
| `PartnerStakedXDC_V3` (per partner) | _discover via registry_ | ERC-4626 vault (native XDC) | Each partner deploys their own pool. Identical runtime bytecode (15% fee constant) so the registry can verify it by `codehash`. |

---

## Multi-chain: psXDC OFT bridge + rate oracle

psXDC bridges to Base, Arbitrum, and BNB Chain via LayerZero V2. The
[bridge](xdc-liquid-staking/bridge.md) uses a lockbox on XDC; the
[rate oracle](xdc-liquid-staking/psxdc-rate-oracle.md) publishes the psXDC/XDC
exchange rate to each chain for lending integrations.

| Contract | Chain | Address | Notes |
| --- | --- | --- | --- |
| `PsxdcOFTAdapter` | XDC | `0xefbb71078cba6425EB6fd957ac1F8a235502D71a` | Lockbox: locks/unlocks real psXDC. One canonical adapter for the whole mesh. |
| `PsxdcRateSender` | XDC | `0x7215e3a0a1F385127eAC1E84D2F30dBF00c76b04` | Broadcasts `previewRedeem(1e18)` to all destination oracles. Permissionless `push()`. |
| `PsxdcOFT` | Base | `0x98D916F5773Ac0482b49856f2659d6c32114C4Ba` | Mint/burn psXDC representation. |
| `PsxdcOFT` | Arbitrum | `0x98D916F5773Ac0482b49856f2659d6c32114C4Ba` | Same address (deployed at matching nonce). |
| `PsxdcOFT` | BNB Chain | `0x98D916F5773Ac0482b49856f2659d6c32114C4Ba` | Same address. |
| `PsxdcRateOracle` | Base | `0x2927630dfDd66433DbA9370b316EF5a8408d5dD2` | `AggregatorV3Interface` psXDC/XDC rate for money markets. |
| `PsxdcRateOracle` | Arbitrum | `0x2927630dfDd66433DbA9370b316EF5a8408d5dD2` | Same address. |
| `PsxdcRateOracle` | BNB Chain | `0x2927630dfDd66433DbA9370b316EF5a8408d5dD2` | Same address. |

Every pathway is verified by four required DVNs (Canary, LayerZero Labs,
Horizen, Nethermind) — the same operator set that services the live XDC lanes.
LayerZero endpoint IDs: XDC 30365, Base 30184, Arbitrum 30110, BNB Chain 30102.

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

- `PrimeStakedXDC_V3_2` (`0xDc74…c734`) to stake, withdraw, redeem, or read the share price.
- `PrimeStakedXDC_V3_2MigrationBridge` (`0x313e…c280`), only if your users hold V2 psXDC and want to migrate.
- `ReferralRegistry` (`0x9765…16B3`), only if you want a first stake to bind a referrer (`stakeWithReferral`).
- `XdcNftStakingVault` (proxy, `0x9f38…4da8`) for any NFT-side action.
- `XdcNftMigratorV2` (`0x69DE…2ea8`), only for migrating legacy V2 NFTs.

The harvester, bypass facet, distributor, vault implementation, and ProxyAdmin are operated by the protocol and do not need to be called from partner code.

→ [V3 Architecture](xdc-liquid-staking/v3-architecture.md) → [Smart Contract Reference (psXDC v3)](xdc-liquid-staking/smart-contract-functions.md) → [Smart Contract Reference (XDC NFT v3)](xdc-staking-nfts/smart-contract-functions.md)
