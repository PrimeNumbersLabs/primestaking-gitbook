# Risk & Compliance

PrimeStaking maintains a comprehensive risk framework and compliance posture designed for institutional partners and regulated environments.

***

## Risk Framework

### Smart Contract Risk

| Risk                   | Severity | Mitigation                                                                                              |
| ---------------------- | -------- | ------------------------------------------------------------------------------------------------------- |
| Contract vulnerability | High     | Independent external audits (QuillAudits - 98.8% score on liquid staking; Nethermind Security on custody / V3 surface), reentrancy guards |
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

| Module                              | Auditor     | Score | Status       |
| ----------------------------------- | ----------- | ----- | ------------ |
| **XDC Staking Contract (liquid)**   | QuillAudits | 98.8% | Published    |
| **Custody / V3 surface**            | Nethermind  | -     | Under review |
| **XDC NFT V3 stack (vault, migrator, harvester, bypass facet)** | Nethermind | - | In scope of the V3 review |

A Nethermind Security audit covering the V3 surface (custody, NFT staking vault, migrator, boost harvester, and legacy bypass facet) is in progress. Results will be published upon completion.

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
