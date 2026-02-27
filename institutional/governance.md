# Governance

PrimeStaking's governance model ensures that protocol upgrades, treasury operations, and critical decisions are made transparently and with appropriate safeguards.

---

## Important Distinction

Governance applies to **contract upgrades and protocol parameters** - not to validator key custody or user fund management. Those are handled by trustless, on-chain smart contracts with no human involvement.

→ [Custody Model](custody-model.md) for details on asset custody

---

## Governance Structure

| Component | Implementation |
| --- | --- |
| **Contract upgrades** | Multisig approval required |
| **Timelock** | Mandatory delay between approval and execution |
| **Treasury management** | Multisig-controlled with on-chain transparency |
| **Emergency actions** | Multisig can pause contracts immediately; patches follow standard upgrade flow |

---

## Multisig

All critical operations require approval from multiple authorized signers. Multisig configuration details are available upon request as part of the due diligence process.

| Parameter | Detail |
| --- | --- |
| **Transparency** | All multisig transactions are on-chain and verifiable |
| **Key management** | Hardware wallet-secured keys held by distinct individuals |

---

## Timelock

Contract upgrades and parameter changes are subject to a mandatory timelock:

| Parameter | Detail |
| --- | --- |
| **Purpose** | Allows community and partner review before changes take effect |
| **Emergency bypass** | Available for critical security patches with full multisig approval |

---

## Upgrade Process

1. **Proposal** - Engineering team proposes a contract upgrade or parameter change
2. **Review** - Internal security review + partner notification (advance notice for material changes)
3. **Multisig approval** - Required signers approve the transaction
4. **Timelock** - Mandatory waiting period before execution
5. **Execution** - Change is deployed on-chain
6. **Verification** - Post-deployment validation and monitoring

---

## Treasury

| Parameter | Detail |
| --- | --- |
| **Controlled by** | Multisig wallet |
| **Funding sources** | Protocol fees from staking operations |
| **Use of funds** | Validator operations, development, audits, ecosystem growth |
| **Transparency** | All treasury transactions on-chain |

---

## Decision-Making Framework

| Decision Type | Process |
| --- | --- |
| **Routine operations** | Team execution, no governance overhead |
| **Contract upgrades** | Multisig + timelock |
| **Fee changes** | Multisig + timelock + partner notification |
| **Emergency pause** | Multisig (immediate, no timelock) |
| **New validator onboarding** | Smart contract-managed (permissionless model) |
| **Partnership agreements** | Team + legal review |

---

## What This Means for Partners

- **No unilateral changes** - protocol modifications require multi-party approval
- **Advance notice** - partners are informed before material changes take effect
- **Verifiable on-chain** - all governance actions are transparent and auditable
- **Predictable operations** - timelock ensures changes are not applied unexpectedly
