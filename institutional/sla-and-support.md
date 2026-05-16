# SLA & Support

PrimeStaking provides differentiated support and uptime commitments based on the partner's integration model.

---

## Uptime SLA

| Component | Target | Measurement |
| --- | --- | --- |
| **Smart contracts** | 99.9% | On-chain availability (dependent on XDC Network uptime) |
| **Validator operations** | 99.5% | Validator uptime and reward generation |
| **API / SDK** | 99.5% | Availability of off-chain integration endpoints |
| **Frontend (Powered by Prime)** | 99.0% | Widget / embedded UI availability |

Uptime is measured monthly. Downtime due to XDC Network-level issues is excluded from SLA calculations.

---

## Incident Response

| Severity | Response Time | Resolution Target |
| --- | --- | --- |
| **Critical** (fund loss risk, contract exploit) | < 1 hour | Pause + patch within 24 hours |
| **High** (degraded service, delayed rewards) | < 4 hours | Resolution within 48 hours |
| **Medium** (non-critical bug, UI issue) | < 24 hours | Resolution within 72 hours |
| **Low** (cosmetic, documentation) | < 48 hours | Next scheduled release |

---

## Support Tiers

### White Label Partners (Model A)

| Feature | Included |
| --- | --- |
| **Dedicated partner manager** | Yes |
| **Technical integration support** | Yes (during onboarding + ongoing) |
| **Priority incident response** | Yes |
| **Custom SLA negotiation** | Available |
| **Monthly performance review** | Yes |
| **Slack / Telegram channel** | Dedicated private channel |

### Powered by Prime Partners (Model B)

| Feature | Included |
| --- | --- |
| **Integration support** | Yes (during onboarding) |
| **Standard incident response** | Yes |
| **Email support** | Yes |
| **Monthly reporting** | Standard automated reports |

---

## Monitoring & Reporting

| Capability | Description |
| --- | --- |
| **On-chain monitoring** | Real-time tracking of staking, rewards, and withdrawal events |
| **Validator health dashboard** | Uptime, reward rate, and performance metrics |
| **Monthly settlement reports** | TVL, rewards generated, fees, revenue share |
| **Incident post-mortems** | Published within 7 days of any critical incident |

---

## Upgrade Policy

- The **psXDC v3 vault is non-upgradeable** — its logic can never be modified. The same holds for the V3 migration bridge, NFT collection, NFT migrator, and boost harvester. Any change to these contracts requires deploying new ones and migrating.
- Only the **`XdcNftStakingVault`** is upgradeable (TransparentUpgradeableProxy controlled by the protocol multisig, with ERC-7201 namespaced storage). Implementation changes go through **multisig governance + delayed handover**.
- Parameter changes on the psXDC v3 vault (loss caps, role rotations, governance delay) follow the on-chain **delayed governance** pattern: schedule → wait `governanceDelay` → execute. Partners can monitor pending changes via on-chain events.
- Partners are notified **at least 7 days** before any material NFT vault upgrade or psXDC governance change.
- Emergency pauses (vault, migrator, harvester) are immediate and require multisig approval. Resuming requires a separate multisig `unpause` call.
- Upgrade history and changelogs are published in documentation.

---

## Contact

For support inquiries:

- **Email:** [admin@primenumbers.xyz](mailto:admin@primenumbers.xyz)
