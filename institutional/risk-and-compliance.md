# Risk & Compliance

PrimeStaking maintains a comprehensive risk framework and compliance posture designed for institutional partners and regulated environments.

***

## Risk Framework

### Smart Contract Risk

| Risk                   | Severity | Mitigation                                                                                              |
| ---------------------- | -------- | ------------------------------------------------------------------------------------------------------- |
| Contract vulnerability | High     | Independent external audits (QuillAudits - 98.8% score on V1 liquid staking; Nethermind Security NM-0843 on V3 vault + V3 Migration Bridge, all Critical/High/Medium findings Fixed), reentrancy guards, emergency pause on the V3 vault |
| Upgrade error          | Medium   | psXDC v3 vault is **non-upgradeable**; only the NFT staking vault is upgradeable, and only via multisig + delayed governance |
| Dependency failure     | Medium   | Minimal external dependencies; core logic is self-contained                                             |
| Economic attack        | Medium   | Buffer + FIFO queue design, bounded scans, per-report and per-day loss caps                             |
| Migration risk         | Low      | One-shot atomic migration with `minSharesOut` slippage protection; migration window is gated; failure modes always revert |

### Validator Risk

| Risk               | Severity | Mitigation                                                                                                                                   |
| ------------------ | -------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Validator downtime | Medium   | Multi-validator delegation, automated failover monitoring                                                                                    |
| Slashing           | **None** | XDC Network does not implement slashing - this is a structural advantage over ETH-based liquid staking protocols where slashing risk is real |
| Reward rate change | Low      | Dynamic APY calculation; transparent communication to partners                                                                               |

### Network Risk

| Risk               | Severity | Mitigation                                                      |
| ------------------ | -------- | --------------------------------------------------------------- |
| XDC Network halt   | Low      | Protocol pauses automatically; no loss of funds                 |
| Fork / chain split | Low      | Protocol follows canonical chain; manual intervention if needed |
| Congestion         | Low      | Transaction prioritization; gas optimization in contracts       |

### Operational Risk

| Risk                 | Severity | Mitigation                                                     |
| -------------------- | -------- | -------------------------------------------------------------- |
| Key compromise       | High     | On-chain smart contract custody - no human key access          |
| Unauthorized upgrade | High     | Multisig + timelock governance                                 |
| Team dependency      | Medium   | Open-source contracts; protocol operates autonomously on-chain |

***

## Audit History

| Module                              | Auditor             | Findings / Score | Status       |
| ----------------------------------- | ------------------- | ---------------- | ------------ |
| **XDC Staking Contract (V1 liquid)**| QuillAudits         | 98.8% score      | Published    |
| **psXDC V3 vault + V3 Migration Bridge** (NM-0843) | Nethermind Security | 21 findings (1C / 2H / 1M / 6L / 9I / 2BP) — **18 Fixed / 3 Acknowledged** | **Published — May 8, 2026** |
| **XDC NFT V3 stack** (vault, collection, migrator, harvester, bypass facet) | Internal — Nethermind planned | 46 unit tests + 17-test audit-fix regression battery | Internal review complete; external review planned |

**Nethermind Security NM-0843 — XDC Prime Stake** (final report, May 08, 2026) covered `PrimeStakedXDC_V3.sol` and `PrimeStakedXDC_V3MigrationBridge.sol` (1,391 LoC). All Critical, High, and Medium findings are Fixed. The three Acknowledged findings are operationally mitigated (loss caps for risk-manager front-running, pre-deployment seed enforcement, and two-call workaround for partial-fill queue redemption).

[→ Read the full NM-0843 report (PDF)](../NM_0843_xdc_prime_stake_FINAL_updated_tests.pdf)

All audit reports are published publicly. Target: **>= 95% score** on every audit, with findings of Medium severity or higher resolved within **72 hours**.

→ [Full Audit Reports](../audits-1/)

***

## Compliance Posture

### Protocol Level

* **Non-custodial** - PrimeStaking never takes custody of user funds
* **Permissionless** - no KYC/AML at the protocol level (open smart contracts)
* **Transparent** - all operations verifiable on-chain
* **Jurisdiction-agnostic** - smart contracts operate globally without geographic restriction

### Partner Level

Partners integrating PrimeStaking are responsible for:

* KYC/AML compliance in their jurisdiction
* Sanctions screening for their users
* Tax reporting and regulatory filings
* Data privacy (GDPR, CCPA, etc.) for their user base

PrimeStaking provides the technical infrastructure; regulatory compliance is handled by the partner at the integration layer.

***

## Incident Response

| SLA                        | Target                                                   |
| -------------------------- | -------------------------------------------------------- |
| **Critical vulnerability** | Pause contracts within 1 hour; patch within 24 hours     |
| **Medium severity issue**  | Assess within 4 hours; resolve within 72 hours           |
| **Low severity issue**     | Assess within 24 hours; resolve in next scheduled update |

→ [SLA & Support](sla-and-support.md)

***

## Liability Framework

### In Case of a Bug or Exploit

* PrimeStaking contracts are audited but not guaranteed to be vulnerability-free
* In the event of an exploit, the protocol will pause operations, assess damage, and work to recover funds
* Partners should carry their own insurance and implement user-facing risk disclosures

### In Case of Delayed Withdrawals

* In V3, withdrawals settle **instantly** when the vault buffer covers them; otherwise they enter the on-chain FIFO queue and settle as new deposits, reward inflows, or masternode resignations replenish liquidity.
* For redemptions that depend on a masternode resignation, the upper bound is the XDC Network's `candidateWithdrawDelay` — approximately **35 days** under typical real-world block times (longer under network congestion).
* PrimeStaking does not guarantee specific withdrawal timelines — the queue is FIFO and depends on network conditions and protocol-level cash flow.
* Partners should communicate **both paths** (instant-when-possible and queued-with-self-claim) to their users.

### In Case of Reward Rate Changes

* APY is variable and depends on validator performance and network conditions
* PrimeStaking communicates material changes to partners with reasonable notice
* Historical reward data is available on-chain for forecasting

→ [Full Disclaimer](../audits-1/disclaimer.md)
