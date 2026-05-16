# Governance

PrimeStaking V3 splits operational, risk, and governance responsibilities across **five roles** with mandatory delayed execution on every sensitive change. The psXDC vault itself is non-upgradeable; only the NFT staking vault is upgradeable, and only through a delayed-governance path.

---

## What governance can — and cannot — do

| Action | Possible under governance? |
| --- | --- |
| Mint psXDC to anyone | **No** — `mint()` is disabled in V3 |
| Withdraw user XDC from the vault | **No** — there is no `ownerWithdraw` function |
| Upgrade `PrimeStakedXDC_V3` | **No** — non-upgradeable, deployed with a regular constructor |
| Upgrade `XdcNftStakingVault` implementation | Yes — TransparentUpgradeableProxy controlled by the protocol multisig with delayed handover |
| Rotate operational / risk roles | Yes — via the delayed-governance path |
| Change loss caps (`maxLossBpsPerReport`, `maxDailyLossBps`) | Yes — schedule → wait `governanceDelay` → execute |
| Pause vault / migrator / harvester | Yes — `PAUSER_ROLE` (multisig) can pause immediately |
| Bypass the time-lock | **No** — direct `grantRole`/`revokeRole`/`renounceRole` are disabled to prevent bypass |

→ [Custody Model](custody-model.md) for details on what the validator and asset layer guarantees.

---

## Role Separation (psXDC v3 vault)

| Role | Holder | Scope |
| --- | --- | --- |
| `DEFAULT_ADMIN_ROLE` | Protocol multisig | Master switch; schedules and executes delayed role/risk changes |
| `OPERATIONS_MANAGER_ROLE` | Designated operations multisig/manager | `setBufferBps`, scan-limit tuning, auto-propose config, masternode parameter tuning |
| `RISK_MANAGER_ROLE` | Designated risk operator | `reportValidatorLoss` (bounded by per-report and per-day caps) |
| `PROPOSER_ROLE` | Designated proposer(s) | `proposeMasternode`, `reportMasternodeResignPrincipal` |
| `MIGRATION_MANAGER_ROLE` | Migration manager | Opens/closes the V2→V3 migration window, tops up backing liquidity via `fundMigrationLiquidity` |

No single key can both move funds and modify roles. Role rotations themselves require delayed execution.

---

## Delayed Governance — schedule → wait → execute

Every sensitive change in `PrimeStakedXDC_V3` follows the same pattern:

| Schedule | Execute (after `governanceDelay`) | Cancel |
| --- | --- | --- |
| `setGovernanceDelay(delay_)` | `executeGovernanceDelay()` | `cancelGovernanceDelayChange()` |
| `setOperationsManager(account)` | `executeOperationsManager()` | `cancelOperationsManagerChange()` |
| `setRiskManager(account)` | `executeRiskManager()` | `cancelRiskManagerChange()` |
| `setMaxLossBpsPerReport(bps)` | `executeMaxLossBpsPerReport()` | `cancelMaxLossBpsPerReportChange()` |
| `setMaxDailyLossBps(bps)` | `executeMaxDailyLossBps()` | `cancelMaxDailyLossBpsChange()` |

Ownership handoff uses the same pattern: `scheduleOwnerTransfer(newOwner)` → wait → `executeOwnerTransfer()` (or `cancelOwnerTransfer()`). `transferOwnership` and `renounceOwnership` are **disabled** so ownership can never change without the delay.

`governanceDelay` itself is bounded between `MIN_GOVERNANCE_DELAY` and `MAX_GOVERNANCE_DELAY` (default 1 day; min 1 minute, max 30 days).

---

## Upgrade Policy

| Component | Upgrade path |
| --- | --- |
| `PrimeStakedXDC_V3` | **None** — non-upgradeable. Replacing the vault requires deploying a new contract and migrating. |
| `PrimeStakedXDC_V3MigrationBridge` | **None** — non-upgradeable. Treasury operations are delayed and capped. |
| `XdcStakedNFT` (collection) | **None** — non-upgradeable. |
| `XdcNftMigrator` | **None** — non-upgradeable. |
| `XdcNftBoostHarvester` | **None** — non-upgradeable. |
| `XdcNftStakingVault` (implementation) | TransparentUpgradeableProxy. Implementation changes are executed by the proxy admin, which is owned by the protocol multisig. ERC-7201 namespaced storage prevents accidental slot collisions on future upgrades. |
| `LegacyMigratorBypassFacet` | Replacement requires a new `diamondCut` on the legacy Diamond (multisig). |

---

## Treasury & Bridge controls

The V3 migration bridge has its own delayed-governance and rate-limit machinery:

| Control | Detail |
| --- | --- |
| Excess treasury withdrawal | Two-step: `withdrawExcessNative(recipient, amount)` (schedule) → `executeExcessNativeWithdrawal()` (after delay). Cancel any time with `cancelExcessNativeWithdrawal()`. |
| Daily outflow guard | `setDailyWithdrawalCap(amount)` bounds total daily outflows. |
| Owner handoff | Two-step delayed transfer, same shape as the vault. |

---

## Pause / Emergency

| Surface | Pause role | Effect |
| --- | --- | --- |
| `PrimeStakedXDC_V3` | `PAUSER_ROLE` (multisig) | Halts stake/redeem flows for incident response |
| `XdcNftStakingVault` | `PAUSER_ROLE` (multisig) | Halts stake/withdraw/claim; `notifyBoost` is **intentionally** still allowed so boost flow continues during ops windows |
| `XdcNftMigrator` | `PAUSER_ROLE` (multisig) | Halts new V2→V3 NFT migrations |
| `XdcNftBoostHarvester` | `PAUSER_ROLE` (multisig) | Halts new boost pushes |

Pauses are immediate (no delay) so the multisig can react to incidents. **Resuming** requires a multisig `unpause()` call; no parameter changes happen during a pause beyond what the underlying role functions allow.

---

## Decision-Making Framework

| Decision Type | Process |
| --- | --- |
| Routine masternode propose / resign | `PROPOSER_ROLE` execution; bounded by on-chain limits |
| Buffer / scan-limit tuning | `OPERATIONS_MANAGER_ROLE` execution |
| Loss caps / role rotations | Schedule → wait `governanceDelay` → execute |
| NFT vault upgrade | Multisig proxy admin call, after audit + partner notification |
| Migration window open / close | `MIGRATION_MANAGER_ROLE` |
| Emergency pause | Multisig (immediate) |
| New audit / partner agreement | Team + legal review |

---

## What this means for Partners

- **No unilateral changes** — every sensitive change is publicly scheduled before it can take effect.
- **Predictable execution** — `governanceDelay` is on-chain; partners can monitor pending changes through events without privileged access.
- **No admin path to user funds** — the V3 vault's design (no `mint`, no `ownerWithdraw`, no upgrade) means governance literally cannot move staker XDC.
- **Auditability** — all role grants, schedules, executions, and parameter updates emit events indexed by the public subgraph.

→ [Custody Model](custody-model.md) → [Architecture Overview](architecture.md) → [Risk & Compliance](risk-and-compliance.md)
