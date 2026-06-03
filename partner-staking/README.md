---
description: White-label XDC liquid staking pools that partners deploy and fully self-manage, powered by the PrimeStaking app. Each pool charges a 15% protocol fee to PrimeStaking.
---

# Partner Staking

{% hint style="warning" %}
**Status: pre-launch.** The Partner Staking contracts (`PartnerStakedXDC_V3` and `PartnerVaultRegistry`) are built and tested but not yet deployed to XDC Mainnet, and the directory is gated behind a feature flag in the app (`NEXT_PUBLIC_PARTNERS_ENABLED`, off by default). Mainnet addresses will be published here at launch. The mechanics below reflect the finalized contracts.
{% endhint %}

**Partner Staking** lets a community, validator, exchange, or institution run its **own** XDC liquid staking pool — its own branded token, its own masternode operators, its own admin keys — while plugging into the PrimeStaking app, UI, and tooling. PrimeStaking does not custody the pool or operate its validators; it provides the audited vault design and the directory, and earns a flat **15% protocol fee** on the pool's staking rewards.

Each pool is a deployment of [`PartnerStakedXDC_V3`](smart-contract-reference.md) — a self-contained, fee-bearing copy of the PrimeStaking flagship [`PrimeStakedXDC_V3`](../xdc-staking/xdc-nfts-staking-system-vaults/xdc-liquid-staking/v3-architecture.md) vault. It is a fully independent contract: separate state, separate token, separate admin keys, separate masternode operators. It shares no storage, funds, or permissions with the flagship vault.

---

## Partner Staking vs. Institutional Integration

PrimeStaking offers two different partner tracks. Don't confuse them:

| | **Partner Staking** (this section) | **Institutional Integration** ([For Partners](../institutional/README.md)) |
| --- | --- | --- |
| What it is | The partner **deploys and runs their own vault** (`PartnerStakedXDC_V3`) and lists it in the PrimeStaking directory | The partner **integrates the flagship PrimeStaking vault** into their own product (White Label / Powered by Prime) |
| Who custodies validators | The **partner's** masternode operators | PrimeStaking's infrastructure |
| Token | The partner's own branded share token (e.g. `acXDC`) | psXDC (the flagship share token) |
| PrimeStaking's cut | Flat **15% protocol fee** skimmed on-chain from rewards | Negotiated revenue share (see [Revenue Model](../institutional/revenue-model.md)) |
| Best for | Communities, validators, regional platforms that want their own pool | Exchanges and custodians embedding XDC staking into an existing app |

---

## Why it's built this way

- **Self-service & self-managed.** The partner deploys the vault, holds the admin keys, registers their own masternode operators, and covers their own masternode hosting costs. PrimeStaking never holds the keys or the funds.
- **Non-custodial & ERC-4626.** Same share-based, native-XDC design as the flagship V3 vault — instant withdrawals against a liquidity buffer, an automatic FIFO queue when the buffer is empty, masternode delegation, time-locked governance, and per-report / daily loss caps.
- **Trust-minimized listing.** The fee rate (15%) and recipient (PrimeStaking treasury) are **compile-time constants**, so every genuine partner vault shares identical runtime bytecode. The [`PartnerVaultRegistry`](registry-and-verification.md) only lists a vault whose `codehash` matches an allow-listed canonical hash — a partner cannot deploy a 0%-fee fork and have it appear in the PrimeStaking UI.
- **Curated visibility.** Registration is open, but the app shows only pools PrimeStaking has marked **Verified**, so the branded directory can never be flooded with spam pools.

---

## How a pool comes to life

1. **Deploy** a `PartnerStakedXDC_V3` with a name, symbol, the XDC validator contract, and a minimum masternode stake. The deployer becomes the vault admin. → [Deploy & List a Pool](deploy-and-list.md)
2. **Configure** operators, KYC, buffer, and governance parameters from the partner admin dashboard.
3. **Register** the vault in `PartnerVaultRegistry` (requires the canonical codehash and the vault admin role), then set presentation metadata. → [Registry & Verification](registry-and-verification.md)
4. **Get verified** by PrimeStaking to earn the "Verified by PrimeStaking" badge and appear in the default directory.
5. **Users stake** native XDC into the pool and earn rewards via NAV growth, net of the 15% protocol fee. → [How It Works](how-it-works.md)

---

## In this section

| Page | What it covers |
| --- | --- |
| [How It Works](how-it-works.md) | The economic model, the 15% fee chokepoint, what the partner manages vs. what PrimeStaking provides |
| [Deploy & List a Pool](deploy-and-list.md) | Deploying `PartnerStakedXDC_V3`, roles & governance, registering and getting verified |
| [Registry & Verification](registry-and-verification.md) | `PartnerVaultRegistry` — codehash gating, verification badge, metadata, delisting |
| [Smart Contract Reference](smart-contract-reference.md) | Function-level reference for both partner contracts |

**Contact:** [admin@primenumbers.xyz](mailto:admin@primenumbers.xyz)
