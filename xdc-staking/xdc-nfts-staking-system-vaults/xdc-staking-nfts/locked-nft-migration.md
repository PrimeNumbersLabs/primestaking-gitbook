# Locked NFTs & Legacy Diamond Bypass

Locked legacy XDC NFTs cannot be burnt-and-redeemed through the standard V2 façade — the `tokenLocked` flag blocks the burn. Without intervention, every locked NFT would be unmigratable until its lock expires.

The V3 stack solves this with a tiny facet — [`LegacyMigratorBypassFacet`](../contract-addresses.md) — added to the legacy Diamond via `diamondCut`. The facet exposes a single mutator that only the V3 migrator can call, which clears the diamond's `tokenLocked` flag so the standard `burnAndRedeem` succeeds inside the same atomic migration transaction. The diamond custodies the psXDC backing every NFT and pays the redemption from its own reserve — the facet makes **no** external call.

{% hint style="info" %}
The bypass facet is **live** on the legacy Diamond. The current facet is `0x6441…25C6`, bound to the current migrator `XdcNftMigratorV2` (`0x36Fe…f026`). Locked migrations work end-to-end through [`/xdc-nfts/migrate`](https://primestaking.xyz/xdc-nfts/migrate) with no extra steps required from the user.
{% endhint %}

{% hint style="warning" %}
**Why the facet was updated.** There are three generations of contracts: the original **v2 staker** (`PrimeStakerV2XDC` `0x2204…B293`), the **legacy Diamond** (the system we migrate *from*), and **V3**. Some Diamond NFTs were themselves migrated v2 → Diamond while locked and carry a historical `lockedFromV2 == true` marker. The original facet treated that marker as *"the funds are still in v2, pull them back via `primeV2.burnToRedeem`."* On-chain inspection (June 2026) showed that is false: the v2 staker has been **drained to ~0**, so `burnToRedeem` reverted, while the Diamond already holds the psXDC. The current facet **removes the v2-staker call entirely** — it just enforces any still-active v2 unlock window, clears `tokenLocked`, and lets the Diamond pay.
{% endhint %}

---

## End-to-end flow (locked NFT)

```
User           Migrator             Old Façade      Legacy Diamond (with bypass facet)        Vault          psXDC v3
 │                │                       │                  │                                    │                │
 │ migrate ─────►│                       │                  │                                    │                │
 │                │ ownerOf, getNFTData ─►│                  │                                    │                │
 │                │ transferFrom(user→me)─►                  │                                    │                │
 │                │ try claim(tokenId) ──►│                  │   // best-effort: folds pending v2 rewards into staked
 │                │ migratorPrepareForBurn(asset, id) ────────►│  // enforces any live v2 unlock window, then clears tokenLocked
 │                │ burnAndRedeem(id)  ──►│                  │                                    │                │
 │                │  …bridge → v3 shares (via PrimeStakedXDC_V3MigrationBridge)…                                    │
 │                │ mintAndStakeLocked(user, id, rarity, shares, lockEnd, lockBoost) ────────────►│                │
 │                │ MigratedLocked(user, id, …) ⏎                                                  │                │
 │ ◄──── user owns new tokenId with original v2 lockEnd preserved on the v3 vault ────────────────┘
```

All the standard migration guarantees still hold: same `tokenId`, atomic execution, no reward forfeiture, no custodial risk. The only additional step is the `migratorPrepareForBurn` call.

---

## What `migratorPrepareForBurn` actually does

The facet has exactly one mutator:

```solidity
function migratorPrepareForBurn(address asset, uint256 tokenId) external;
```

- **Caller restriction**: only the configured migrator (`XdcNftMigratorV2`) can call it. The migrator address is baked into the facet at deployment.
- **Effect**: clears the `tokenLocked[asset][tokenId]` flag in the legacy Diamond's storage. If the NFT is `lockedFromV2`, it **first** checks the real v2 `unlockTimestamp` and reverts `V2NftStillLocked` if the lock is still active; otherwise it clears the flag. It makes **no** external call (no `primeV2.burnToRedeem`) — the Diamond already custodies the psXDC and pays it on `burnAndRedeem`.
- **No new privileges**: the facet does not enable anything else — once `tokenLocked` is cleared, the legacy `burnAndRedeem` flow runs normally.

Two informational view selectors are also added in the same cut so the migrator (and indexers) can introspect lock state without storage tricks:

| Function | Returns |
| --- | --- |
| `isMigratorBypassNeeded(address asset, uint256 tokenId)` | true if the migrator needs to call `migratorPrepareForBurn` before burning |
| `lockedFromV2UnlockTimestamp(address asset, uint256 tokenId)` | the real `lockedFromV2` unlock time (reads storage directly, not via the legacy view) |

---

## The legacy `getNFTData` caveat

This is the most important thing to know if you're auditing or debugging the bypass:

> The legacy `StakerGetterFacet.getNFTData(asset, tokenId)` view function **synthesises** the returned struct's `lockedData.lockedFromV2` field from the **separate** `tokenLocked[asset][tokenId]` mapping. It does **not** report the storage `lockedFromV2` flag.

So a token whose view-returned `lockedFromV2 == true` may actually have been locked via `lockNFT` (storage `lockedFromV2 == false`). The facet reads the **real** storage flag via `LegacyAppStorageMirror`, so its behaviour is independent of the façade aliasing:

- For both lock origins it **clears `tokenLocked`** so the burn succeeds.
- For genuine storage-`lockedFromV2` tokens it **additionally enforces the original v2 unlock window** (reverting `V2NftStillLocked` if the lock has not yet expired).
- In **neither** case does it touch the v2 staker — the Diamond already holds the underlying psXDC and pays it on `burnAndRedeem`.

Indexers and integrators that need to know the actual lock origin should use the facet's view functions instead of the legacy `getNFTData` field.

---

## What happens on the V3 side

Once the legacy burn succeeds the migrator continues exactly as for an unlocked NFT — bridge the redeemed psXDC into V3 shares, then call:

```solidity
mintAndStakeLocked(
  address to,
  uint256 tokenId,
  uint8   rarity,
  uint256 shares,
  uint64  lockEnd,    // copied from the legacy NFT
  uint256 lockBoost   // configured on the V3 vault
);
```

`lockEnd` is the **original** V2 unlock timestamp. This is the key property: a user cannot dodge the lock by routing through the migrator. On the V3 side, the NFT remains locked (`withdraw`, `merge`, `burnAndRedeem` revert) until the same time it would have unlocked on V2.

---

## What if I migrate a locked NFT *before* `lockEnd`?

That's the supported case — the migrator preserves the lock and the V3 vault honours it. You won't be able to `withdraw`, `merge`, or `burnAndRedeem` your V3 NFT until `lockEnd` passes, but you will:

- Earn the boost slice on the locked weight (which is higher than the unlocked weight) the entire time.
- Earn the base NAV through the underlying psXDC v3 shares.
- Be able to `claim` the boost at any time during the lock.

After `lockEnd` you can call `unlock(tokenId)` on the V3 vault to remove the lock bonus from the weight, or leave it locked indefinitely to keep the boost slice.

---

## Failure modes (handled by revert)

| Condition | Revert |
| --- | --- |
| Migrator deployed but bypass facet not yet cut into the Diamond | `LegacyDiamondRequiredForLockedNft(tokenId)` — the user keeps the legacy NFT |
| A genuine `lockedFromV2` NFT whose original v2 lock has **not** yet expired | `V2NftStillLocked(tokenId, unlockTs)` — the facet refuses to clear the lock early; user keeps the legacy NFT until the lock expires |
| Slippage exceeded on the V3 bridge | Whole migration reverts, user keeps the legacy NFT |

Every failure mode is "fail closed": the user's legacy NFT remains in place. No mid-state outcomes.

---

## Live deployment

| Component | Address |
| --- | --- |
| Legacy Diamond | [`0x7a5d364b97126600C0AdDFD5C339230748bcaA17`](https://xdcscan.com/address/0x7a5d364b97126600C0AdDFD5C339230748bcaA17) |
| Bypass facet (**live**) | [`0x64413bAD206b5D90a5010cc683F50086407F25C6`](https://xdcscan.com/address/0x64413bAD206b5D90a5010cc683F50086407F25C6) |
| Migrator (bound to bypass facet, **live**) | [`0x36Fe37Ca1FEF0e409977a1c28d191B55333cf026`](https://xdcscan.com/address/0x36Fe37Ca1FEF0e409977a1c28d191B55333cf026) |
| Bypass facet (original, superseded) | [`0x275641d5bA81786A7e60352F990F0c203e7D1836`](https://xdcscan.com/address/0x275641d5bA81786A7e60352F990F0c203e7D1836) |
| Migrator (original, paused) | [`0x45e2e91098A8451EA450754784e043bb3F8C7dFb`](https://xdcscan.com/address/0x45e2e91098A8451EA450754784e043bb3F8C7dFb) |

The original cut was executed by the legacy Diamond's `defaultAdmin` (the protocol multisig). The remediation (June 2026) was rolled out as a `diamondCut` **Replace** swapping the original facet's `migratorPrepareForBurn` implementation for the v2-staker-free version above, a new `XdcNftMigratorV2` with id-remapping, the corresponding role swap (granting `MIGRATOR_ROLE`/`MINTER_ROLE` to the new migrator and revoking the old), and pausing the original migrator. The cut adds only `migratorPrepareForBurn(address,uint256)` (with `isMigratorBypassNeeded` and `lockedFromV2UnlockTimestamp` as informational view selectors). Storage layout is unaffected; no existing selector was modified.

→ [Migrate XDC NFTs to V3](migrate-nfts-v2-to-v3.md) → [Smart Contract Reference (V3)](smart-contract-functions.md) → [Deployed Contracts & Addresses](../contract-addresses.md)
