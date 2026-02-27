# Custody Model

PrimeStaking operates a **non-custodial, smart contract-based** custody model. Validator keys and staked assets are managed entirely by audited on-chain contracts — no human interaction in custody flows.

---

## Design Principles

| Principle | Implementation |
| --- | --- |
| **Non-custodial** | Users retain full ownership of assets at all times |
| **Permissionless** | Anyone can verify validator state and staked assets on-chain |
| **Trustless** | No single entity controls validator keys — the protocol enforces custody rules through code |
| **Transparent** | All custody operations are logged on the blockchain |

---

## How It Works

### Validator Key Management

- Validator keys are generated and secured by on-chain smart contracts
- No human operator has direct access to private keys
- Key rotation and management are governed by contract logic

### Staked Asset Custody

- User XDC deposits are held in the Liquid Staking Pool contract
- The contract delegates staked XDC to validators programmatically
- Withdrawal requests are processed through the contract's redemption queue
- At no point does a human operator have discretionary control over user funds

---

## Institutional Considerations

| Question | Answer |
| --- | --- |
| **Who controls the masternodes?** | Smart contracts manage validator operations programmatically |
| **Who signs upgrades?** | Multisig governance with timelock (see [Governance](governance.md)) |
| **Is there a multisig?** | Yes — contract upgrades require multi-party approval |
| **Is there a timelock?** | Yes — upgrade execution is delayed to allow community review |
| **Who controls the treasury?** | Protocol treasury is governed by multisig with transparent on-chain operations |
| **Is there automated reporting?** | Yes — all staking, reward, and withdrawal events are indexed on-chain |

---

## Audit & Collaboration

The custody model is developed in collaboration with:

- **XDC Core team** — network-level validator integration
- **Nethermind** — smart contract development and security review
- **QuillAudits** — independent external audit (98.8% score on staking contracts)

A dedicated audit of the custody contracts is in progress. Once complete, the system transitions to a fully **permissionless, trustless** model with zero operational trust requirements.

---

## Risk Mitigation

| Risk | Mitigation |
| --- | --- |
| **Smart contract exploit** | Independent audits, reentrancy guards, pausable contracts |
| **Validator downtime** | Multi-validator delegation, performance monitoring |
| **Unauthorized upgrades** | Multisig + timelock governance |
| **Key compromise** | On-chain key management — no human access to private keys |
| **Delayed withdrawals** | Transparent queue processing with predictable timelines (~31 days) |

---

## What This Means for Partners

- **No third-party custodian risk** — assets are secured by code, not by an institution
- **Verifiable at any time** — on-chain state is the single source of truth
- **No operational dependency** — the protocol operates autonomously once deployed
- **Institutional-grade transparency** — full auditability aligned with exchange compliance requirements
