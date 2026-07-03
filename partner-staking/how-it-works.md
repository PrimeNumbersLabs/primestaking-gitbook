# How Partner Staking Works

A partner pool is an independent ERC-4626 liquid staking vault. Stakers deposit native XDC and receive the pool's share token; the share price (NAV) grows as the pool's masternodes earn validator rewards. The only economic difference from the flagship [`PrimeStakedXDC_V3_1`](../xdc-staking/xdc-nfts-staking-system-vaults/xdc-liquid-staking/v3-architecture.md) vault is a **15% protocol fee** that is skimmed from rewards before they reach stakers.

---

## The 15% protocol fee

| Parameter | Value |
| --- | --- |
| `PLATFORM_FEE_BPS` | `1500` (**15%**) |
| `PLATFORM_FEE_RECIPIENT` | [`0x1658b14Cb483B96E7e10227fF00f438858089127`](https://xdcscan.com/address/0x1658b14Cb483B96E7e10227fF00f438858089127) (PrimeStaking treasury) |

Both are **compile-time constants** — not constructor arguments. That is deliberate: it means every genuine partner vault compiles to **identical runtime bytecode**, which is what lets the [`PartnerVaultRegistry`](registry-and-verification.md) verify a pool by its `codehash`. Changing the fee or recipient would require a recompile and a fresh allow-list entry, so a partner can't quietly run a 0%-fee fork and still get listed.

The fee applies only to **rewards** (yield), never to principal:

- Staker deposits and returned masternode principal are **not** taxed.
- Only the growth of managed assets above the tracked NAV — i.e. validator rewards — is subject to the 15%.

### Path-independent fee chokepoint

The fee is not charged on a single "rewards in" function that a partner could route around. Instead, every balance-changing operation reconciles through one internal chokepoint (`_syncTrackedAssetsExcludingInflow`). Whenever that reconciliation detects that the pool's assets have grown beyond the tracked NAV (excluding deposits and principal returns), it:

1. Computes the surplus (the yield),
2. Skims 15% of it to the PrimeStaking treasury, and
3. Credits the remaining 85% to stakers via NAV appreciation.

```
validator rewards arrive  ─►  reconcile assets  ─►  surplus detected
                                                        │
                                   ┌────────────────────┴───────────────────┐
                                   ▼                                         ▼
                       15% skimmed to PrimeStaking            85% credited to stakers (NAV ↑)
```

Because of this design the fee is unavoidable: force-sending XDC (`selfdestruct` / direct transfer), routing rewards around the `receive()` path, or delivering yield through a masternode-resignation are all taxed identically on the next reconciliation. The recipient is a protocol-controlled address that always accepts native XDC, so the skim cannot be made to revert to grief stakers.

---

## Who does what

| Responsibility | Owner |
| --- | --- |
| Deploy the vault & hold admin keys | **Partner** |
| Register & run masternode operators | **Partner** |
| Cover masternode hosting / infrastructure cost | **Partner** |
| Set buffer, min stake, KYC, governance parameters | **Partner** |
| Vault contract design (audited flagship V3 logic) | PrimeStaking |
| App, UI, directory & verification badge | PrimeStaking |
| Receives the 15% protocol fee | PrimeStaking |

PrimeStaking provides the infrastructure and the storefront; the partner runs the pool. PrimeStaking is **non-custodial** with respect to partner pools — it never holds a pool's keys or funds and only ever receives the on-chain protocol fee.

---

## What stakers experience

Identical to flagship psXDC, minus the 15% reward haircut:

- **Stake** native XDC via `stake` / `depositNative`, receive the pool's share token at the current exchange rate.
- **NAV growth** — no claim button; shares simply become worth more XDC as net (post-fee) rewards accrue.
- **Withdraw** — instant if the liquidity buffer covers it (`withdraw` / `redeem`), otherwise the request enters an automatic FIFO queue (`withdrawWithQueue` / `redeemWithQueue`) and is claimed once masternode payouts return. Queued requests can be cancelled before they're processed.
- **Same safety rails** as flagship V3: time-locked governance changes, per-report and daily validator-loss caps, pausable, reentrancy-guarded, and role-separated operations.

→ [Deploy & List a Pool](deploy-and-list.md) → [Registry & Verification](registry-and-verification.md) → [Smart Contract Reference](smart-contract-reference.md)
