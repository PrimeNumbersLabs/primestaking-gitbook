# Registry & Verification

`PartnerVaultRegistry` is the on-chain directory that powers the PrimeStaking app's partner pool listing. It answers one question trust-minimally: *"is this address a genuine, unmodified partner vault that charges the 15% protocol fee, controlled by the person registering it?"*

It is an `Ownable2Step` contract (two-phase ownership handoff so control can't be lost to a mistyped address). The owner is PrimeStaking.

---

## Codehash gating

Because `PartnerStakedXDC_V3` bakes the fee rate and recipient in as constants, every genuine partner vault has **identical runtime bytecode** and therefore one canonical `codehash`. PrimeStaking allow-lists that hash:

```solidity
setCanonicalCodeHash(bytes32 codeHash, bool allowed)   // owner only, once per published vault bytecode version
```

`register(vault)` then enforces:

- `isCanonicalCodeHash[vault.codehash] == true` — else `NotCanonicalBytecode`
- caller holds `vault`'s `DEFAULT_ADMIN_ROLE` — else `NotVaultAdmin`
- `vault != address(0)` and not already registered — else `ZeroAddress` / `AlreadyRegistered`

This makes it impossible to list a fork with a different fee, a different recipient, or hidden modifications. If the vault bytecode is ever revised, PrimeStaking publishes and allow-lists the new codehash; older deployments stay valid under their own hash unless explicitly revoked.

---

## Verification & visibility

| State | Meaning | Shown in default directory? |
| --- | --- | --- |
| Registered | Passed codehash + admin checks | No (reachable by direct address) |
| Verified | PrimeStaking has vetted the pool (`setVerified(vault, true)`) | **Yes** — carries the "Verified by PrimeStaking" badge |
| Unregistered | Never listed, or delisted via `unregister` | No |

The app reads `verifiedVaults()` for the curated listing, so the directory can never be flooded with unverified spam pools, while registration itself stays permissionless. `setVerified` and `unregister` are **owner-only** (PrimeStaking); `unregister` clears all registry state for a vault but does not touch the vault contract — its admin can register it again later.

---

## Metadata

Presentation data lives in the registry (not the vault, whose bytecode is codehash-locked) and is editable only by the vault's admin:

```solidity
struct PoolMeta { string description; string website; string logoURI; string twitter; string telegram; }
setMetadata(address vault, PoolMeta meta)   // vault DEFAULT_ADMIN_ROLE only
```

---

## Read API (used by the app & indexers)

| Function | Returns |
| --- | --- |
| `allVaults()` | Every registered vault address |
| `verifiedVaults()` | Only the verified subset (the default directory) |
| `vaultsLength()` / `vaultAt(uint256)` | Enumerate the global list |
| `vaultsByAdmin(address)` | Pools registered by a given admin |
| `isRegistered(address)` / `isVerified(address)` | Status flags |
| `registrantOf(address)` | Who registered a vault |
| `metadata(address)` | The `PoolMeta` for a vault |

## Events

| Event | When |
| --- | --- |
| `CanonicalCodeHashSet(codeHash, allowed)` | A canonical bytecode hash is allow-listed or revoked |
| `VaultRegistered(vault, registrant, name, symbol)` | A pool is added to the directory |
| `VaultVerifiedSet(vault, verified)` | A pool's verified badge is toggled |
| `VaultUnregistered(vault, registrant)` | A pool is delisted |
| `MetadataUpdated(vault, admin)` | A pool's presentation metadata changes |

→ [How It Works](how-it-works.md) → [Deploy & List a Pool](deploy-and-list.md) → [Smart Contract Reference](smart-contract-reference.md)
