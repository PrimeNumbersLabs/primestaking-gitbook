# Deploy & List a Pool

Bringing a partner pool online is a two-stage process: **deploy** your own `PartnerStakedXDC_V3`, then **register** it in the `PartnerVaultRegistry` so it appears in the PrimeStaking app. Both stages are self-service from the partner dashboard.

{% hint style="info" %}
Live on XDC Mainnet: the registry ([`0x325D…3a67`](https://xdcscan.com/address/0x325DEEA5C7c0Ce0D774c4A67EcCaAf1cF8953a67)) and its canonical codehash allow-list are deployed, and pools are already registering through the flow below.
{% endhint %}

---

## 1. Deploy the vault

`PartnerStakedXDC_V3` is deployed with a regular constructor:

```solidity
constructor(
    string  name_,            // ERC-20 name shown in wallets/UI, e.g. "Acme Staked XDC"
    string  symbol_,          // ERC-20 symbol, e.g. "acXDC"
    address initialValidator, // the XDC validator contract the vault delegates to
    uint256 initialMinStake   // minimum masternode proposal size
) payable
```

The deploying address (`msg.sender`) becomes the vault **admin** and is also granted the proposer, operations-manager, and risk-manager roles. Any XDC sent with deployment is seeded as the initial stake.

To stay eligible for listing, deploy the **canonical, unmodified** `PartnerStakedXDC_V3` bytecode that PrimeStaking has allow-listed. The fee rate and recipient are constants baked into that bytecode, so a custom build will not match the registry's codehash and cannot be listed.

---

## 2. Configure the pool

From the partner admin dashboard (or directly against the contract), the admin sets the pool up:

| Action | Function | Role |
| --- | --- | --- |
| Register a masternode operator | `addOperator(address)` / `removeOperator(address)` | `DEFAULT_ADMIN_ROLE` |
| Submit pool KYC to the validator | `submitKYC(string)` | `DEFAULT_ADMIN_ROLE` |
| Set liquidity buffer (≤ 50%) | `setBufferBps(uint256)` | `OPERATIONS_MANAGER_ROLE` |
| Set minimum masternode stake | `setMinStake(uint256)` | `OPERATIONS_MANAGER_ROLE` |
| Set / change validator | `setValidator(address)` | `OPERATIONS_MANAGER_ROLE` |
| Configure auto-propose | `setAutoProposeConfig(bool,uint256)` | `OPERATIONS_MANAGER_ROLE` |
| Pause / unpause | `pause()` / `unpause()` | `DEFAULT_ADMIN_ROLE` |

Masternode proposals require the vault itself to be KYC-verified on the validator, and only registered operators can be proposed. Auto-propose round-robins across operators once the available balance crosses the threshold.

---

## 3. Governance & roles

The pool ships with the same hardened governance model as the flagship V3 vault:

- **Direct role and ownership changes are disabled.** `grantRole`, `revokeRole`, `renounceRole`, `transferOwnership`, and `renounceOwnership` all revert. Privileged changes must go through the **schedule → wait → execute** flow.
- **Time-locked changes.** Changing the proposer, operations manager, risk manager, loss caps, the governance delay itself, or transferring admin ownership are each two-phase: `set…` schedules the change, and after `governanceDelay` has elapsed `execute…` applies it. Pending changes can be cancelled.
- **Governance delay**: default **1 day**, configurable between **1 minute** and **30 days** (itself time-locked).
- **Loss caps**: `reportValidatorLoss` (RISK_MANAGER only) is bounded by a per-report cap (default **10%**) and a rolling daily cap (default **20%**), both expressed in bps of tracked assets.

| Role | Powers |
| --- | --- |
| `DEFAULT_ADMIN_ROLE` | Operators, KYC, pause, schedule/execute all governance changes |
| `PROPOSER_ROLE` | Propose / resign masternodes |
| `OPERATIONS_MANAGER_ROLE` | Buffer, min stake, validator, auto-propose, scan limits |
| `RISK_MANAGER_ROLE` | Report validator losses (within caps) |

---

## 4. Register in the directory

Once deployed and configured, list the pool so it shows up in PrimeStaking:

```solidity
PartnerVaultRegistry.register(address vault)
```

`register` succeeds only if **both** of the following hold:

1. The vault's `codehash` is an **allow-listed canonical hash** (`isCanonicalCodeHash[vault.codehash] == true`), proving it's a genuine, unmodified `PartnerStakedXDC_V3` that charges the 15% fee.
2. The caller **holds the vault's `DEFAULT_ADMIN_ROLE`**, proving they actually control the pool.

It then records the registrant and emits `VaultRegistered(vault, registrant, name, symbol)`.

### Set presentation metadata

The vault's admin can attach off-chain-style presentation data (kept in the registry because the vault bytecode is codehash-locked and must not change):

```solidity
PartnerVaultRegistry.setMetadata(address vault, PoolMeta { description, website, logoURI, twitter, telegram })
```

---

## 5. Get verified

Registration is **open**, but visibility is **curated**. By default the app reads `verifiedVaults()` and shows only pools PrimeStaking has marked verified:

```solidity
PartnerVaultRegistry.setVerified(address vault, bool verified)   // PrimeStaking (registry owner) only
```

Verified pools earn the **"Verified by PrimeStaking"** badge and appear in the default directory. Unverified pools remain reachable by direct address, so a pool is usable the moment it's registered, but the branded directory can't be spammed. PrimeStaking can also `unregister` an abusive or abandoned pool entirely (the vault contract itself is untouched; it can be re-registered later by its admin).

→ [How It Works](how-it-works.md) → [Registry & Verification](registry-and-verification.md) → [Smart Contract Reference](smart-contract-reference.md)
