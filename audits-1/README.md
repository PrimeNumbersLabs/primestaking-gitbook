# Audits

## Security Audits

All PrimeStaking smart contracts undergo independent external audits before deployment and whenever significant updates are introduced.

***

### Methodology

1. **Reputable external auditor** - We partner with leading firms (e.g., QuillAudits) that review every line of code and validate the economic logic of the contracts.
2. **Two-phase process**
   * _Preliminary report:_ Identification of findings.
   * _Fix & Verify:_ The engineering team resolves issues and the auditor performs final verification.
3. **Transparent disclosure** - Full reports are published so the community can verify the scope and applied fixes.

***

### Published Reports

| Module                                                                                                                                 | Auditor             | Findings                                          | Link                                                                                                                                                                                                                            |
| -------------------------------------------------------------------------------------------------------------------------------------- | ------------------- | ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [**XDC Staking Contract (V1 / liquid staking)**](https://www.quillaudits.com/leaderboard/prime-numbers/prime-numbers-staking-contract) | QuillAudits         | 98.8% score                                       | [Report](https://www.quillaudits.com/leaderboard/prime-numbers/prime-numbers-staking-contract)                                                                                                                                  |
| **psXDC V3 vault + V3 Migration Bridge** (NM-0843)                                                                                     | Nethermind Security | 1 Critical · 2 High · 1 Medium · 6 Low · 9 Info · 2 Best Practices — **18 Fixed / 3 Acknowledged** | [Report (PDF)](../NM_0843_xdc_prime_stake_FINAL_updated_tests.pdf) |

{% embed url="https://www.quillaudits.com/leaderboard/prime-numbers/prime-numbers-staking-contract" %}

> Our target is to keep every audit score **>= 95%** and close findings of Medium severity or higher within **72 hours**.

***

### Nethermind Security — NM-0843, XDC Prime Stake (May 08, 2026)

**Scope (1,391 LoC):**

- `PrimeStakedXDC_V3.sol` — ERC-4626 native-XDC vault, non-upgradeable, with the buffer / FIFO queue / `claimQueuedAssets` flow, masternode round-robin auto-propose, delayed governance, role split, and per-report / per-day loss caps.
- `PrimeStakedXDC_V3MigrationBridge.sol` — V2 psXDC → V3 share migration with `minSharesOut` slippage protection, time-locked excess withdrawals, daily withdrawal cap, and delayed owner transfer.

**Process:** initial review (commit `f92b803`, March 26, 2026) → fix verification → final commit `2d97d9b` (May 08, 2026). Documentation Assessment: Medium. Test Suite Assessment: Medium.

**Findings, by severity and status:**

| Severity     | Count | Fixed | Acknowledged |
| ------------ | ----- | ----- | ------------ |
| Critical     | 1     | 1     | 0            |
| High         | 2     | 2     | 0            |
| Medium       | 1     | 1     | 0            |
| Low          | 6     | 5     | 1            |
| Informational | 9    | 8     | 1            |
| Best Practices | 2   | 2     | 0            |
| **Total**    | **21** | **18** | **3**       |

**Acknowledged findings (no functional fix shipped):**

| Finding | Severity | Why acknowledged |
| --- | --- | --- |
| Users can frontrun `reportValidatorLoss(...)` to evade slashing penalties | Info | Mitigated operationally via the per-report and per-day loss caps; routing all exits through the queue would degrade UX. |
| Constructor does not enforce a non-zero seed at deployment | Low | Operational pre-deployment requirement; the deployment script seeds the vault. |
| `redeemWithQueue` / `withdrawWithQueue` use all-or-nothing logic (no partial fill) | Info | Two-call pattern (`redeem(maxRedeem)` + `redeemWithQueue(rest)`) gives the same outcome; partial fill considered for a future revision. |

**All Critical, High, and Medium findings are Fixed.** Highlights:

- *Critical* — returned masternode stakes were double-counted in `trackedTotalAssets`; resolved by routing principal returns through `resignMasternode(...)` instead of the unprivileged `receive()` path.
- *High* — receivers could trigger a gas-bomb in `_processWithdrawalQueueInternal(...)` to brick the queue; resolved by enforcing a strict gas limit on the external call.
- *High* — permissionless `syncTrackedAssets()` during migration could let early migrators steal funded liquidity; resolved by gating `syncTrackedAssets` with `whenMigrationFinished`.
- *Medium* — deferred queued payouts were not ring-fenced from the protocol's accounting; resolved by introducing a `totalDeferredPayouts` accumulator and excluding it from `_availableForImmediateWithdrawal`, `_availableForQueuePayout`, and `_syncTrackedAssetsExcludingInflow`.

The full per-finding write-up, recommendations, and per-fix verification is in the [PDF report](../NM_0843_xdc_prime_stake_FINAL_updated_tests.pdf).

***

### Other V3 stack components

The XDC NFT V3 surface (`XdcStakedNFT`, `XdcNftStakingVault`, `XdcNftMigrator`, `XdcNftBoostHarvester`, `LegacyMigratorBypassFacet`) ships with an internal 46-test suite plus a 17-test audit-fix regression battery covering every C/H/M finding from the internal review. External audit of this surface is planned and any future report will be published on this page.

***

**Note:** The core PRFI Token contract was fully audited. Although not part of this staking documentation, the report is publicly available.
