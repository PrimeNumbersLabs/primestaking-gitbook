---
description: Reference material for users still on V2 contracts. Not recommended for new users — migrate to V3.
---

# Legacy (V2 — historical)

The V2 stack remains operational so users who have not migrated can continue to use the product. Every new deposit, the official UI, and all partner integrations are wired against the **V3** contracts described in the main documentation. The pages in this section exist for historical context and to support users finishing their migration.

{% hint style="warning" %}
New users should always use V3. The V2 contracts described here do not implement self-service withdrawals, share-price reward accrual, or the boost accumulator, and are not part of the active reward roadmap.
{% endhint %}

---

## Legacy contract addresses

| Contract | Address | Role |
| --- | --- | --- |
| Legacy psXDC v2 (proxy) | [`0x9B8e12b0BAC165B86967E771d98B520Ec3F665A6`](https://xdcscan.com/address/0x9B8e12b0BAC165B86967E771d98B520Ec3F665A6) | V2 psXDC ERC-20 token. Use this address when approving the [V2 → V3 migration bridge](../xdc-staking/xdc-nfts-staking-system-vaults/xdc-liquid-staking/staking-guide/migration.md). |
| Legacy Diamond (ERC-2535) | [`0x7a5d364b97126600C0AdDFD5C339230748bcaA17`](https://xdcscan.com/address/0x7a5d364b97126600C0AdDFD5C339230748bcaA17) | Hosts all legacy XDC NFT facets. Now also hosts [`LegacyMigratorBypassFacet`](../xdc-staking/xdc-nfts-staking-system-vaults/xdc-staking-nfts/locked-nft-migration.md) so locked NFTs can be migrated in one transaction. |
| Legacy ERC-721 façade | [`0x9D458330e458f11fd1cE7E44B3a66568af8076a0`](https://xdcscan.com/address/0x9D458330e458f11fd1cE7E44B3a66568af8076a0) | The user-visible NFT contract behind V2 XDC NFTs. |

---

## How V2 still behaves

| Surface | V2 behaviour |
| --- | --- |
| Liquid staking deposit | Mints V2 psXDC at a fixed 1:1 ratio with XDC. |
| Liquid staking rewards | Distributed via the legacy `notifyRewardAmount` flow; users manually claim from the legacy Rewards tab. |
| Liquid staking withdrawal | Submitted as a request and processed via the validator queue under the legacy operational flow. |
| XDC NFTs | Continue to function under the V2 reward model (monthly reward pool driven by NFT multipliers). |
| Migration | Both psXDC v2 → V3 shares and legacy NFTs → V3 NFTs are available; see the V3 documentation for current migration UX. |

There is no plan to deprecate V2 forcibly — users can take their time migrating.

---

## Pages in this section

- [Historical: pstXDC → psXDC migration](pstxdc-migration.md) — the original migration from the **pstXDC** token to **psXDC** (V1 → V2). Closed out years ago; kept here for any historical user still on pstXDC.

→ [Migrate V2 psXDC → V3 (current)](../xdc-staking/xdc-nfts-staking-system-vaults/xdc-liquid-staking/staking-guide/migration.md) → [Migrate XDC NFTs to V3 (current)](../xdc-staking/xdc-nfts-staking-system-vaults/xdc-staking-nfts/migrate-nfts-v2-to-v3.md)
