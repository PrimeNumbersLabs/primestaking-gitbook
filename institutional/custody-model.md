# Custody Model

PrimeStaking operates a **non-custodial, smart contract-based** custody model. Validator keys and staked assets are managed entirely by audited on-chain contracts - no human interaction in custody flows.

---

## Design Principles

| Principle | Implementation |
| --- | --- |
| **Non-custodial** | Users retain full ownership of assets at all times |
| **Permissionless** | Anyone can verify validator state and staked assets on-chain |
| **Trustless** | No single entity controls validator keys - the protocol enforces custody rules through code |
| **Transparent** | All custody operations are logged on the blockchain |

---

## How It Works

### Validator Key Management

- Validator keys are generated and secured by on-chain smart contracts
- No human operator has direct access to private keys
- Key rotation and management are governed by contract logic

### Staked Asset Custody

- User XDC deposits are held in the [`PrimeStakedXDC_V3`](../xdc-staking/xdc-nfts-staking-system-vaults/contract-addresses.md) vault — **non-upgradeable**, so the logic that holds your XDC can never be modified
- The vault keeps a tunable **liquid buffer** (default 5% of total assets); excess is auto-delegated to KYC-verified masternode operators on the XDC Network
- Withdrawal requests use `redeemWithQueue` — instant when the buffer covers them, queued FIFO otherwise. Failed payouts defer into `pendingQueuedAssets` and the user collects them via `claimQueuedAssets`. There is no admin approval step at any point.
- The vault has **no `mint` and no `ownerWithdraw`** — these were removed in V3. Even the protocol admin cannot move user funds.

---

## Two Distinct Layers

It is important to distinguish between **asset custody** and **contract governance**:

| Layer | Mechanism | Human Involvement |
| --- | --- | --- |
| **Validator key custody** | Fully on-chain, smart contract-managed | None - trustless by design |
| **psXDC v3 vault** | Non-upgradeable — deployed with a regular constructor, no proxy | None - cannot be modified |
| **NFT staking vault upgrades** | TransparentUpgradeableProxy with ERC-7201 namespaced storage, controlled by the protocol multisig with delayed governance | Yes - multi-party approval required for vault implementation changes |
| **Parameter changes (psXDC v3)** | Role-gated + delayed governance (schedule → wait → execute) | Yes - multi-party approval, but cannot move user funds |

User funds are secured by code with zero human access. The psXDC vault itself cannot be upgraded. Only the NFT staking vault is upgradeable, and only through the multisig + delayed-governance path.

→ [Governance Details](governance.md)

---

## Institutional Considerations

| Question | Answer |
| --- | --- |
| **Who controls the masternodes?** | Smart contracts manage validator operations programmatically |
| **Who signs upgrades?** | Multisig governance with timelock (see [Governance](governance.md)) |
| **Is there a multisig?** | Yes - contract upgrades require multi-party approval |
| **Is there a timelock?** | Yes - upgrade execution is delayed to allow review |
| **Who controls the treasury?** | Protocol treasury is governed by multisig with transparent on-chain operations |
| **Is there automated reporting?** | Yes - all staking, reward, and withdrawal events are indexed on-chain |

---

## Audit & Collaboration

The custody model is developed in collaboration with:

- **Nethermind** - smart contract development and security review
- **XDC Core team** - network-level validator integration
- **QuillAudits** - independent external audit (98.8% score on staking contracts)

A dedicated audit of the custody contracts is in progress. Full results will be published upon completion.

---

## Risk Mitigation

| Risk | Mitigation |
| --- | --- |
| **Smart contract exploit** | Independent audits, reentrancy guards, pausable contracts |
| **Validator downtime** | Multi-validator delegation, performance monitoring |
| **Unauthorized upgrades** | psXDC v3 vault is **non-upgradeable** (cannot happen); NFT vault upgrades require multisig + delayed governance |
| **Key compromise** | On-chain key management - no human access to private keys |
| **Delayed withdrawals** | Transparent FIFO queue; instant when the buffer permits, otherwise bounded by the network's `candidateWithdrawDelay` (~35 days under typical block times) |

---

## What This Means for Partners

- **No third-party custodian risk** - assets are secured by code, not by an institution
- **Verifiable at any time** - on-chain state is the single source of truth
- **No operational dependency** - the protocol operates autonomously once deployed
- **Institutional-grade transparency** - full auditability aligned with exchange compliance requirements
