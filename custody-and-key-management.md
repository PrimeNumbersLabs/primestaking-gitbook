# Custody & Key Management

PrimeStaking uses a **permissionless, smart contract-based** validator custody model - eliminating human interaction from custody flows entirely.

***

## How It Works

* **Smart contract-based execution** - validator keys and staked XDC are managed entirely by audited smart contracts, with no human interaction in custody flows.
* **Permissionless** - anyone can verify the state of validators and staked assets on-chain. No centralized approval required.
* **Trustless** - no single entity controls the keys. The protocol enforces custody rules through code, not operational trust.
* **Institutional-grade transparency** - full operational transparency aligned with institutional security standards.

***

## Audit & Collaboration

PrimeStaking's custody infrastructure is developed in collaboration with:

* Nethermind — smart contract development and security review
* XDC Core team — network-level validator integration

An independent security audit of the custody contracts is underway as part of our ongoing security program. Full results will be published upon completion.

***

## What This Means for Users

| Property                 | Detail                                            |
| ------------------------ | ------------------------------------------------- |
| **Custody model**        | Smart contract-based (no third-party custodian)   |
| **Key management**       | Validator keys secured by on-chain contracts      |
| **Human interaction**    | Eliminated from custody flows                     |
| **Verifiability**        | Fully transparent and auditable on the blockchain |
| **User action required** | None - fully seamless                             |

***

## Impact on Users

* Users retain **full ownership** of their assets at all times.
* Smart contract-based custody eliminates reliance on any centralized custodian, strengthening the protocol's decentralization.
* In V3, **withdrawal UX has improved**: redemptions settle instantly when the vault's liquid buffer permits, and otherwise enter a permissionless FIFO queue users can [self-claim](xdc-staking/xdc-nfts-staking-system-vaults/xdc-liquid-staking/staking-guide/withdrawals-instant-vs-queued.md) (`claimQueuedAssets`). The upper bound for queue settlement is the XDC Network's `candidateWithdrawDelay` — approximately **35 days** under typical block times.
* The psXDC v3 vault is **non-upgradeable**, so the contract that holds your XDC cannot be modified by anyone. Only the NFT staking vault is upgradeable, and only through multisig + delayed governance.
