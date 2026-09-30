# Smart Contract Reference (psXDC V3)

Technical reference for [`PrimeStakedXDC_V3_2`](../contract-addresses.md), the V3 liquid staking vault. The contract is an ERC-4626-style native-XDC vault with self-service withdrawals, a buffer-aware queue, on-chain masternode integration, and time-locked governance.

{% hint style="info" %}
Address: [`0xDc74c0DaED82ae94486DeeF22991d2F54173c734`](https://xdcscan.com/address/0xDc74c0DaED82ae94486DeeF22991d2F54173c734). **Non-upgradeable**: deployed with a regular constructor, no proxy.
{% endhint %}

---

## User functions

### `stake() payable`

Deposits native XDC and mints psXDC shares to the sender at the current exchange rate.

| Parameter | Detail |
| --- | --- |
| `msg.value` | Amount of native XDC to deposit |
| Returns | psXDC shares minted |

### `depositNative(uint256 assets, address receiver) payable`

Same as `stake` but lets you specify a separate `receiver`. `msg.value` must equal `assets`.

### `withdraw(uint256 assets, address receiver, address owner)` / `redeem(uint256 shares, address receiver, address owner)`

Standard ERC-4626 instant redemption. Reverts if the vault's liquid buffer cannot cover the request.

### `withdrawWithQueue(uint256 assets, address receiver, address owner)` / `redeemWithQueue(uint256 shares, address receiver, address owner)`

The "smart" redemption used by the app. Settles **instantly** when the buffer is sufficient, otherwise **escrows the shares** and adds a request to the FIFO queue.

### `claimQueuedAssets(address receiver)`

Pays out any XDC parked in `pendingQueuedAssets` for the caller (e.g. from previously failed receiver payouts or processed queue requests where the receiver could not receive directly).

### `cancelQueuedWithdrawal(uint256 requestId)`

Cancels a pending queued withdrawal you previously created and returns the escrowed shares to your wallet.

### `migrate(address receiver, uint256 assets, uint256 minSharesOut)` (bridge-only)

Mints V3 shares to `receiver` against `assets` of XDC contributed by the migration bridge. Only callable by the configured `PrimeStakedXDC_V3MigrationBridge` while the migration window is open. `minSharesOut` protects callers from unfavorable exchange-rate movement.

---

## Read-only helpers

| Function | Returns |
| --- | --- |
| `totalAssets()` | Total XDC tracked by the vault (includes principal staked with masternodes) |
| `convertToShares(uint256 assets)` / `convertToAssets(uint256 shares)` | Exchange-rate conversions |
| `previewDeposit` / `previewWithdraw` / `previewRedeem` | UI-friendly previews using the current rate |
| `maxWithdraw(address)` / `maxRedeem(address)` | Liquidity-aware caps that reflect the buffer |
| `outstandingValidatorPrincipalByOperator(address operator)` | Principal currently staked with a specific masternode operator |

---

## Operator functions (validator management)

Anyone with the appropriate role can call these; users do not call them directly.

| Function | Role | Purpose |
| --- | --- | --- |
| `proposeMasternode(...)` | `PROPOSER_ROLE` | Manually propose a new masternode |
| `triggerAutoPropose(uint256 maxNodes)` | anyone | Trigger the bounded auto-propose loop. Always blocked while there is a withdrawal queue backlog so user redemptions are prioritized over new proposals |
| `processWithdrawalQueue(uint256 maxRequests)` | anyone | Process up to `maxRequests` FIFO entries |
| `syncTrackedAssets()` | anyone | Reconcile `trackedTotalAssets` with the contract's native balance after unexpected inflows |

---

## Operations & risk

These are gated by the V3 role split. They never let an admin move user funds; they only tune parameters and report validator outcomes.

| Function | Role | Notes |
| --- | --- | --- |
| `distributeRewards() payable` | `OPERATIONS_MANAGER_ROLE` | Reward lane: credits the sent XDC (minus the optional `rewardFeeBps` skim, default 0, hard-capped at 20%) to `trackedTotalAssets`, i.e. raises the psXDC share price for every holder. This is how the ~4.5% APY reaches holders, roughly monthly. |
| `setRewardFeeConfig(uint16 bps, address recipient)` | `DEFAULT_ADMIN_ROLE` | Optional protocol fee on distributions (currently 0). |
| `fundBacking() payable` | `MIGRATION_MANAGER_ROLE` | Backing lane: adds un-earmarked XDC intended for `proposeMasternode`. Never mints shares or moves NAV. |
| `setOperatorScanLimit(uint256)` / `setQueueScanLimit(uint256)` | `OPERATIONS_MANAGER_ROLE` | Bounded scan limits used by auto-propose and queue processing |
| `setMinStake(uint256)` | `OPERATIONS_MANAGER_ROLE` | Minimum XDC required to attempt a masternode propose |
| `setValidator(address)` | `OPERATIONS_MANAGER_ROLE` | Configures the XDC validator contract address |
| `reportValidatorLoss(address operator, uint256 assets)` | `RISK_MANAGER_ROLE` | Reports a validator loss; capped per-report (`maxLossBpsPerReport`) and per-day (`maxDailyLossBps`) |
| `reportMasternodeResignPrincipal(address operator)` | `PROPOSER_ROLE` | Accounts for masternode principal that has been returned to the vault after resignation cooldown |

`grantRole`, `revokeRole`, and `renounceRole` are intentionally **disabled**. Sensitive role rotations must go through the delayed governance path below to prevent bypassing the time-lock.

---

## Delayed governance

Every sensitive change is a **schedule → wait → execute** flow gated by `governanceDelay`:

| Schedule | Execute | Cancel |
| --- | --- | --- |
| `setGovernanceDelay(delay_)` | `executeGovernanceDelay()` | `cancelGovernanceDelayChange()` |
| `setOperationsManager(account)` | `executeOperationsManager()` | `cancelOperationsManagerChange()` |
| `setRiskManager(account)` | `executeRiskManager()` | `cancelRiskManagerChange()` |
| `setMaxLossBpsPerReport(bps)` | `executeMaxLossBpsPerReport()` | `cancelMaxLossBpsPerReportChange()` |
| `setMaxDailyLossBps(bps)` | `executeMaxDailyLossBps()` | `cancelMaxDailyLossBpsChange()` |

The owner handoff itself uses the same pattern:

- `scheduleOwnerTransfer(newOwner)` → wait `governanceDelay` → `executeOwnerTransfer()`. Cancel any time with `cancelOwnerTransfer()`.

`transferOwnership` and `renounceOwnership` are **disabled** so ownership can never change without the delay.

---

## Migration controls

| Function | Role | Purpose |
| --- | --- | --- |
| `setMigrationBridge(address)` | admin | Configures the address allowed to call `migrate(...)` |
| `setMigrationInProgress(bool)` | `MIGRATION_MANAGER_ROLE` | Opens or closes the migration window |
| `fundBacking()` payable | `MIGRATION_MANAGER_ROLE` | Adds un-earmarked backing XDC (for masternode proposals); a plain XDC transfer from the same role is earmarked to `queueBackingBudget` instead. Neither mints shares nor moves NAV. |

Direct native sends to the vault are routed by sender: from the migration manager they are earmarked to `queueBackingBudget`; from the operations manager or admin they are treated as a reward distribution; from anyone else they are silent backing donations (balance only, never NAV).

---

## V3.2: the permanent ledger model

V3.1 introduced a temporary `underBackedMode` that suspended reward distribution until the vault was fully backed. V3.2 (the live vault) makes the partially-backed state the **permanent, first-class operating model** and removes the mode flag, the percentage buffer (`bufferBps`) and `fundMigrationLiquidity` entirely:

| Surface | Detail |
| --- | --- |
| `trackedTotalAssets` | A **liability ledger**: what the vault owes holders. Real backing (native balance + `outstandingValidatorPrincipal`) may sit below it; the gap is disclosed off-chain from the public getters as `(trackedTotalAssets + totalFailedPayouts) − (balance + outstandingValidatorPrincipal)`. |
| Reward lane — `distributeRewards()` | Always open. The operations manager forwards the period's validator yield; it raises `trackedTotalAssets` (the share price) regardless of the backing gap, so holders earn on schedule. Optional hard-capped `rewardFeeBps` (default 0). |
| Backing lanes — `fundBacking()` and migration-manager sends | Always open, never NAV. `fundBacking()` adds un-earmarked liquidity for `proposeMasternode`. A raw send from the `MIGRATION_MANAGER_ROLE` wallet is earmarked to `queueBackingBudget`, from which the FIFO withdrawal queue is settled **exclusively**. Neither mints shares nor touches `trackedTotalAssets`; third-party sends are accepted as silent backing donations, so outsiders cannot pump the share price. |
| `queueBackingBudget` (public uint) | The ring-fenced queue budget. Also receives returned masternode principal (via `withdrawResignedMasternode`) up to the queue's unfunded amount. |
| Immediate withdrawals | Draw only from **unencumbered surplus liquidity** (balance − queue earmark − failed-payout reserve). Fresh stakers can always exit their own liquidity, and the earmarked queue funding cannot be front-run. |
| Queue processing | `processWithdrawalQueue(maxRequests)` walks the FIFO paying `previewRedeem(shares)` at the current rate from `queueBackingBudget`; a request the budget cannot cover is skipped (and retried later), never cancelled. |
| NAV protection | `syncTrackedAssets()` only ratchets tracked assets **upwards** (donations, vault-node rewards). Real validator losses go through `reportValidatorLoss`, bounded by `maxLossBpsPerReport` / `maxDailyLossBps` under the risk manager. |


---

## Events worth indexing

| Event | When |
| --- | --- |
| `Deposit(sender, owner, assets, shares)` | Stake / deposit |
| `Withdraw(sender, receiver, owner, assets, shares)` | Instant or queued payout |
| `WithdrawalQueued(requestId, owner, shares)` | A request entered the FIFO queue |
| `WithdrawalProcessed(requestId, receiver, assets, shares)` | The queue processed a request |
| `WithdrawalCanceled(requestId, owner, shares)` | User cancelled their own queued request |
| `AutoProposeFailed(reason)` | Auto-propose was attempted but didn't execute; never reverts the stake flow |
| `ValidatorLossReported(operator, assets, reporter)` | Risk manager recorded a loss |
| `MigrationLiquidityFunded(amount)` | Migration manager topped up bridge liquidity |

---

## Security properties

- Non-blocking auto-propose path: stake never reverts because of a failed propose.
- Deferred payout fallback for rejecting receivers via `pendingQueuedAssets` + `claimQueuedAssets`.
- Liquidity-aware `maxWithdraw` / `maxRedeem`.
- Bounded scans (`operatorScanLimit`, `queueScanLimit`) to prevent gas-griefing.
- AccessControl role split (proposer / operations / risk / migration).
- Delayed execution for every sensitive role / risk parameter change.
- Per-report and cumulative per-day loss caps.
- Direct role mutation (`grantRole`, `revokeRole`) disabled to prevent delayed-governance bypass.

→ [V3 Architecture](v3-architecture.md) → [Deployed Contracts & Addresses](../contract-addresses.md)
