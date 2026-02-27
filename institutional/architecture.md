# Architecture Overview

PrimeStaking's infrastructure is designed for reliability, transparency, and institutional-grade integration. All core logic runs on audited, on-chain smart contracts deployed on the XDC Network.

---

## System Design

```
User / Partner API
        │
        ▼
┌──────────────────────┐
│   Frontend / SDK     │  ← Web app, API, or partner integration layer
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│  Liquid Staking      │  ← Core smart contracts (XDC Network)
│  Contract Suite      │
│  ┌────────────────┐  │
│  │ Staking Pool   │──┼──→ Accepts XDC deposits, mints psXDC
│  │ Reward Engine  │──┼──→ Calculates and distributes rewards
│  │ Withdrawal Mgr │──┼──→ Processes redemption queue
│  └────────────────┘  │
└──────┬───────────────┘
       │
       ▼
┌──────────────────────┐
│  Validator Layer     │  ← XDC Network masternodes
│  (Smart Contract     │
│   Managed)           │
└──────────────────────┘
```

---

## Contract Topology

| Contract | Function | Network |
| --- | --- | --- |
| **Liquid Staking Pool** | Accepts XDC deposits, mints/burns psXDC | XDC Network |
| **Reward Distribution** | Proportional reward allocation to psXDC holders | XDC Network |
| **Withdrawal Queue** | Manages redemption requests through validator unstaking | XDC Network |
| **Validator Custody** | Smart contract-based validator key management (permissionless) | XDC Network |
| **XDC NFT Staking** | NFT-based staking vaults with multiplier logic | XDC Network |
| **PRFI NFT Staking** | EIP-2535 Diamond proxy with omnichain support (LayerZero) | Base (L2) |

---

## Validator Infrastructure

PrimeStaking operates XDC Network masternodes that generate the staking yield:

- **Validator keys** are managed by on-chain smart contracts - no human custody
- **Staked XDC** is allocated across validators for redundancy
- **Performance monitoring** ensures uptime and optimal reward generation
- **Slashing protection** is built into the operational model

The protocol is transitioning to a fully **permissionless, trustless** validator custody model in collaboration with the XDC Core team and Nethermind.

→ [Custody Model Details](custody-model.md)

---

## Security Layers

| Layer | Implementation |
| --- | --- |
| **Smart contract audits** | QuillAudits - 98.8% score on XDC Staking Contract |
| **Permissionless custody** | Validator keys secured by on-chain contracts |
| **On-chain transparency** | Every staking, reward, and withdrawal event is logged and verifiable |
| **Upgradeable architecture** | Diamond Standard (EIP-2535) for PRFI contracts; proxy patterns for XDC contracts |
| **Reentrancy protection** | Built into all state-changing functions |
| **Pausable** | Emergency pause capability with authorized roles |

---

## Data Flow

### Staking

1. User deposits XDC → Liquid Staking Pool contract
2. Contract mints psXDC (1:1) → User wallet
3. XDC is delegated to validators → Rewards begin accruing

### Reward Distribution

1. Validators generate staking rewards on XDC Network
2. Reward Engine calculates proportional allocation per psXDC holder
3. Users claim rewards from the Rewards contract

### Withdrawal

1. User burns psXDC → Withdrawal Queue contract
2. Contract initiates validator unstaking
3. XDC is returned after validator queue processing (~31 days average)

---

## Integration Points

Partners can integrate at multiple levels:

| Integration Level | Description | Use Case |
| --- | --- | --- |
| **Frontend** | Embed the staking UI or build a custom frontend | White-label web integration |
| **Smart Contract** | Interact directly with staking contracts | Backend/API integration |
| **SDK / API** | Use PrimeStaking's SDK for programmatic access | Exchange integration |
| **Data / Reporting** | On-chain event indexing for portfolio and settlement reporting | Compliance and reconciliation |

→ [Integration Models](integration-models.md)
