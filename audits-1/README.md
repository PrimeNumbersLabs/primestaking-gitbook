# Audits

## Security Audits

All PrimeStaking smart contracts undergo independent external audits before deployment and whenever significant updates are introduced.

***

### Methodology

1. **Reputable external auditor** - We partner with leading firms (e.g., QuillAudits) that review every line of code and validate the economic logic of the contracts.
2. **Two-phase process**
   * _Preliminary report:_ Identification of findings.
   * _Fix & Verify:_ The engineering team resolves issues and the auditor performs final verification.
3. **Transparent disclosure** - Full reports are published so the community can verify the scope and applied fixes.

***

### Published Reports

| Module                                                                                                           | Auditor     | Score | Link                                                                                           |
| ---------------------------------------------------------------------------------------------------------------- | ----------- | ----- | ---------------------------------------------------------------------------------------------- |
| [**XDC Staking Contract (V1 / liquid staking)**](https://www.quillaudits.com/leaderboard/prime-numbers/prime-numbers-staking-contract) | QuillAudits | 98.8% | [Report](https://www.quillaudits.com/leaderboard/prime-numbers/prime-numbers-staking-contract) |

{% embed url="https://www.quillaudits.com/leaderboard/prime-numbers/prime-numbers-staking-contract" %}

> Our target is to keep every audit score **>= 95%** and close findings of Medium severity or higher within **72 hours**.

***

### V3 Audit Scope (in progress)

PrimeStaking has engaged **Nethermind Security** to audit the V3 surface, covering:

| Component | Notes |
| --- | --- |
| `PrimeStakedXDC_V3` | ERC-4626 vault, non-upgradeable, queue mechanics, delayed governance, loss caps |
| `PrimeStakedXDC_V3MigrationBridge` | V2 psXDC → V3 share migration with `minSharesOut` slippage protection |
| `XdcNftStakingVault` | Upgradeable proxy, Synthetix-style boost accumulator, ERC-7201 namespaced storage |
| `XdcStakedNFT` | New ERC-721 NFT collection |
| `XdcNftMigrator` | One-shot atomic V2 → V3 NFT migration, `tokenId` / rarity / lock preservation |
| `XdcNftBoostHarvester` | Boost funding pipe (`feed`, `harvest`, `notifyBoost`) |
| `LegacyMigratorBypassFacet` | Facet added to the legacy Diamond via `diamondCut` for locked-NFT migration |

Nethermind Security is a tier-1 Web3 security firm and trusted auditing partner for Lido, EtherFi, Worldcoin, and Optimism, among others. The full report will be published upon completion.

The V3 stack also includes an internal 46-test suite (audit-fix regressions, locked-NFT migration, queue resilience, role gating, delayed governance, loss caps, bridge timelock / cap / slippage) that runs against every change.

***

**Note:** The core PRFI Token contract was fully audited. Although not part of this staking documentation, the report is publicly available.
