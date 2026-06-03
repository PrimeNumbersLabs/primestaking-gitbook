# Smart Contract Reference

Two contracts make up Partner Staking: **`PartnerStakedXDC_V3`** (the per-partner vault) and **`PartnerVaultRegistry`** (the shared directory).

{% hint style="warning" %}
Pre-launch — no mainnet addresses yet. They'll be added to [Deployed Contracts & Addresses](../xdc-staking/xdc-nfts-staking-system-vaults/contract-addresses.md) at launch.
{% endhint %}

---

## `PartnerStakedXDC_V3` — the partner vault

An ERC-4626, native-XDC, share-based liquid staking vault (`ReentrancyGuard`, `ERC4626`, `Pausable`, `AccessControl`). It is a fee-bearing copy of the flagship `PrimeStakedXDC_V3` with separate state, token, keys, and operators. `asset()` is the zero address because the underlying is native XDC; `totalAssets()` returns `trackedTotalAssets`.

### Constants

| Constant | Value | Meaning |
| --- | --- | --- |
| `PLATFORM_FEE_BPS` | `1500` | 15% protocol fee on reward inflows |
| `PLATFORM_FEE_RECIPIENT` | `0x1658…9127` | PrimeStaking treasury (fee recipient) |
| `DEFAULT_MASTERNODE_STAKE` | `10,000,000 XDC` | Default masternode size |
| `MIN_REWARD_DISTRIBUTION` | `1000 XDC` | Minimum non-privileged reward push via `receive()` |
| `MAX_BUFFER_BPS` | `5000` | Max liquidity buffer (50%) |
| `DEFAULT_MAX_LOSS_BPS_PER_REPORT` | `1000` | 10% per-report validator-loss cap |
| `DEFAULT_MAX_DAILY_LOSS_BPS` | `2000` | 20% rolling daily loss cap |
| `DEFAULT_GOVERNANCE_DELAY` | `1 day` | Default timelock (range 1 min – 30 days) |

### Stake & withdraw

| Function | Notes |
| --- | --- |
| `stake(uint256 assets) payable` | Native-XDC stake; `msg.value` must equal `assets`. Mints shares at the current rate. |
| `depositNative(uint256 assets, address receiver) payable` | Same, crediting `receiver`. |
| `deposit(...)` / `mint(...)` | **Disabled** — revert `NativeDepositRequired` (deposits must be native, via the payable wrappers). |
| `withdraw(uint256 shares)` | Legacy-compatible redeem-by-shares. |
| `withdraw(uint256 assets, address receiver, address owner)` / `redeem(...)` | Standard ERC-4626 exits; revert if the liquid buffer can't cover them. |
| `withdrawWithQueue(uint256 assets, address receiver)` | Instant if liquid, else queues a FIFO request. |
| `redeemWithQueue(uint256 shares, address receiver)` | Same, by shares. |
| `cancelQueuedWithdrawal(uint256 requestId)` | Cancel an unprocessed queued request; shares returned. |
| `processWithdrawalQueue(uint256 maxRequests)` | Anyone can advance the queue once liquidity returns. |
| `claimQueuedAssets(address payable receiver)` | Claim a deferred payout (if a queued transfer failed). |
| `maxWithdraw` / `maxRedeem` | Clamped to what the buffer can pay immediately. |

### Masternodes & operators

| Function | Role |
| --- | --- |
| `addOperator` / `removeOperator` | `DEFAULT_ADMIN_ROLE` |
| `submitKYC(string)` | `DEFAULT_ADMIN_ROLE` |
| `proposeMasternode(address candidate, uint256 amount)` | proposer or admin |
| `triggerAutoPropose(uint256 maxNodes)` | anyone (round-robins operators when funded) |
| `resignMasternode(address)` / `withdrawResignedMasternode(uint256)` | proposer or admin |
| `setValidator` / `setMinStake` / `setBufferBps` / `setAutoProposeConfig` / `setOperatorScanLimit` / `setQueueScanLimit` | `OPERATIONS_MANAGER_ROLE` |
| `reportValidatorLoss(address operator, uint256 assets)` | `RISK_MANAGER_ROLE`, within per-report & daily caps |

### Governance (timelocked, two-phase)

Direct `grantRole` / `revokeRole` / `renounceRole` / `transferOwnership` / `renounceOwnership` all **revert**. Use the scheduled flow instead:

| Schedule | Execute | Cancel |
| --- | --- | --- |
| `setProposer` | `executeProposer` | `cancelProposerChange` |
| `setOperationsManager` | `executeOperationsManager` | `cancelOperationsManagerChange` |
| `setRiskManager` | `executeRiskManager` | `cancelRiskManagerChange` |
| `setMaxLossBpsPerReport` | `executeMaxLossBpsPerReport` | `cancelMaxLossBpsPerReportChange` |
| `setMaxDailyLossBps` | `executeMaxDailyLossBps` | `cancelMaxDailyLossBpsChange` |
| `setGovernanceDelay` | `executeGovernanceDelay` | `cancelGovernanceDelayChange` |
| `scheduleOwnerTransfer` | `executeOwnerTransfer` | `cancelOwnerTransfer` |

Each executes only after `governanceDelay` has elapsed.

### Key views & marker

| Function | Returns |
| --- | --- |
| `isPartnerStakedXDCV3()` | `true` — marker the registry/UI use to identify partner vaults |
| `name()` / `symbol()` | The partner's branded name/symbol |
| `totalAssets()` / `desiredBuffer()` | Tracked NAV and target buffer |
| `isKYCVerified(address)` | Whether an address is KYC'd on the validator |

### Notable events

`Staked`, `Withdrawn`, `WithdrawalQueued` / `WithdrawalQueueProcessed` / `WithdrawalQueueCancelled`, `MasternodeProposed`, `ValidatorLossReported`, and **`PlatformFeeSkimmed(recipient, amount)`** (emitted on every fee skim).

---

## `PartnerVaultRegistry` — the directory

`Ownable2Step`; owner is PrimeStaking. See [Registry & Verification](registry-and-verification.md) for the full write-up.

| Function | Caller | Purpose |
| --- | --- | --- |
| `setCanonicalCodeHash(bytes32, bool)` | owner | Allow-list / revoke a canonical vault bytecode hash |
| `register(address vault)` | vault admin | List a vault (requires canonical codehash + admin role) |
| `setVerified(address, bool)` | owner | Toggle the "Verified by PrimeStaking" badge |
| `unregister(address)` | owner | Delist an abusive/abandoned pool |
| `setMetadata(address, PoolMeta)` | vault admin | Set description / website / logo / socials |
| `allVaults` / `verifiedVaults` / `vaultsByAdmin` / `vaultAt` / `vaultsLength` | view | Enumerate the directory |
| `isRegistered` / `isVerified` / `registrantOf` / `metadata` | view | Per-vault status & data |

Events: `CanonicalCodeHashSet`, `VaultRegistered`, `VaultVerifiedSet`, `VaultUnregistered`, `MetadataUpdated`.

→ [Partner Staking overview](README.md) → [How It Works](how-it-works.md) → [Deploy & List a Pool](deploy-and-list.md)
