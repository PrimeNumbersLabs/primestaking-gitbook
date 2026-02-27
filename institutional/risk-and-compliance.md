# Risk & Compliance

PrimeStaking maintains a comprehensive risk framework and compliance posture designed for institutional partners and regulated environments.

---

## Risk Framework

### Smart Contract Risk

| Risk | Severity | Mitigation |
| --- | --- | --- |
| Contract vulnerability | High | Independent external audits (QuillAudits - 98.8% score), reentrancy guards, formal verification roadmap |
| Upgrade error | Medium | Multisig governance + timelock delay on all contract upgrades |
| Dependency failure | Medium | Minimal external dependencies; core logic is self-contained |
| Economic attack | Medium | Rate limiting, withdrawal queue design, on-chain monitoring |

### Validator Risk

| Risk | Severity | Mitigation |
| --- | --- | --- |
| Validator downtime | Medium | Multi-validator delegation, automated failover monitoring |
| Slashing | Low | XDC Network does not implement slashing (masternode model) |
| Reward rate change | Low | Dynamic APY calculation; transparent communication to partners |

### Network Risk

| Risk | Severity | Mitigation |
| --- | --- | --- |
| XDC Network halt | Low | Protocol pauses automatically; no loss of funds |
| Fork / chain split | Low | Protocol follows canonical chain; manual intervention if needed |
| Congestion | Low | Transaction prioritization; gas optimization in contracts |

### Operational Risk

| Risk | Severity | Mitigation |
| --- | --- | --- |
| Key compromise | High | On-chain smart contract custody - no human key access |
| Unauthorized upgrade | High | Multisig + timelock governance |
| Team dependency | Medium | Open-source contracts; protocol operates autonomously on-chain |

---

## Audit History

| Module | Auditor | Date | Score | Status |
| --- | --- | --- | --- | --- |
| **XDC Staking Contract** | QuillAudits | Apr 2025 | 98.8% | Published |
| **Custody Contracts** | In progress | - | - | Under review |

All audit reports are published publicly. Target: **>= 95% score** on every audit, with findings of Medium severity or higher resolved within **72 hours**.

→ [Full Audit Reports](../audits-1/)

---

## Compliance Posture

### Protocol Level

- **Non-custodial** - PrimeStaking never takes custody of user funds
- **Permissionless** - no KYC/AML at the protocol level (open smart contracts)
- **Transparent** - all operations verifiable on-chain
- **Jurisdiction-agnostic** - smart contracts operate globally without geographic restriction

### Partner Level

Partners integrating PrimeStaking are responsible for:

- KYC/AML compliance in their jurisdiction
- Sanctions screening for their users
- Tax reporting and regulatory filings
- Data privacy (GDPR, CCPA, etc.) for their user base

PrimeStaking provides the technical infrastructure; regulatory compliance is handled by the partner at the integration layer.

---

## Incident Response

| SLA | Target |
| --- | --- |
| **Critical vulnerability** | Pause contracts within 1 hour; patch within 24 hours |
| **Medium severity issue** | Assess within 4 hours; resolve within 72 hours |
| **Low severity issue** | Assess within 24 hours; resolve in next scheduled update |

→ [SLA & Support](sla-and-support.md)

---

## Liability Framework

### In Case of a Bug or Exploit

- PrimeStaking contracts are audited but not guaranteed to be vulnerability-free
- In the event of an exploit, the protocol will pause operations, assess damage, and work to recover funds
- Partners should carry their own insurance and implement user-facing risk disclosures

### In Case of Delayed Withdrawals

- Withdrawal timing depends on the XDC Network validator queue (~31 days average)
- PrimeStaking does not guarantee specific withdrawal timelines
- Partners should communicate expected withdrawal windows to their users

### In Case of Reward Rate Changes

- APY is variable and depends on validator performance and network conditions
- PrimeStaking communicates material changes to partners with reasonable notice
- Historical reward data is available on-chain for forecasting

→ [Full Disclaimer](../audits-1/disclaimer.md)
