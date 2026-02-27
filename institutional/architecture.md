# Architecture Overview

PrimeStaking's infrastructure is designed for reliability, transparency, and institutional-grade integration. All core logic runs on audited, on-chain smart contracts deployed on the XDC Network.

The infrastructure is developed in collaboration with **Nethermind** (smart contract engineering and security) and the **XDC Core team** (network-level validator integration).

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

---

## Validator Infrastructure

PrimeStaking operates XDC Network masternodes that generate the staking yield:

- **Validator keys** are managed by on-chain smart contracts - no human custody
- **Staked XDC** is allocated across validators for redundancy
- **Performance monitoring** ensures uptime and optimal reward generation
- **Zero slashing risk** - XDC Network uses a masternode model that does not implement slashing, eliminating the capital loss risk present in ETH-based liquid staking protocols

→ [Custody Model Details](custody-model.md)

---

## Security Layers

| Layer | Implementation |
| --- | --- |
| **Smart contract audits** | QuillAudits - 98.8% score on XDC Staking Contract |
| **Permissionless custody** | Validator keys secured by on-chain contracts |
| **On-chain transparency** | Every staking, reward, and withdrawal event is logged and verifiable |
| **Upgradeable architecture** | Proxy patterns for controlled contract upgrades |
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

Partners can integrate at multiple levels depending on the chosen model. Technical details are shared during the integration process.

| Integration Level | Description | Use Case |
| --- | --- | --- |
| **Frontend** | Embed the staking UI or build a custom frontend | White-label web integration |
| **Smart Contract** | Interact directly with staking contracts | Backend/API integration |
| **Data / Reporting** | On-chain event indexing for portfolio and settlement reporting | Compliance and reconciliation |

→ [Integration Models](integration-models.md)
