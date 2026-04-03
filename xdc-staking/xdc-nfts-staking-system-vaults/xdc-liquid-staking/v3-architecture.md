# psXDC V3 — Non-Custodial Vault Architecture

{% hint style="warning" %}
V3 is currently undergoing its final security audit and will be deployed soon. The architecture described here is final and reflects the audited codebase.
{% endhint %}

psXDC V3 is a complete rebuild of the liquid staking contract, moving from a custodial model to a fully non-custodial, ERC-4626 tokenized vault. This is the architecture that will underpin all psXDC staking going forward.

---

## Overview

| | |
| --- | --- |
| **Standard** | ERC-4626 (OpenZeppelin tokenized vault) |
| **Token** | psXDC — yield-bearing share token |
| **Pricing** | Exchange rate increases as rewards accrue (not fixed 1:1) |
| **Withdrawals** | Self-service — no admin approval required |
| **Staking** | Direct masternode integration via XDC validator contract |
| **Governance** | Role separation with mandatory time-locked delays |
| **Admin mint/withdraw** | Disabled — not possible in V3 |

---

## How It Works

### 1. Stake

Deposit XDC through the staking interface. The contract mints psXDC shares based on the current exchange rate. The more rewards the vault accumulates, the more XDC each psXDC share is worth.

There is no minimum deposit. Staking is permissionless — no account, no KYC, just connect your wallet.

### 2. Earn Rewards

Your staked XDC is pooled and delegated to KYC-verified masternode operators on the XDC Network. Validator rewards flow back into the contract automatically, increasing the total assets backing all psXDC shares.

You don't need to claim rewards. They are embedded in the share price — your psXDC simply becomes worth more XDC over time.

### 3. Withdraw

You have full self-service access to your XDC:

- **Instant withdrawal** — if the contract has sufficient liquid XDC, you redeem shares and receive XDC in the same transaction. No admin approval. No waiting.
- **Queued withdrawal** — if most XDC is staked in masternodes and liquidity is temporarily low, your request enters an automatic FIFO queue. It processes as soon as liquidity returns from new deposits, reward inflows, or masternode resignations.
- **DEX swap** — sell psXDC directly on [XSWAP](https://info.xspswap.finance/#/pools/0xc4a0b4ce176c623a281bc565bfd35eab4fd7050a) for immediate exit at market rate.

### 4. Use psXDC

psXDC is a standard ERC-20 token on the XDC Network. While your XDC earns yield in the background, you can:

- **Hold** to accumulate rewards passively
- **Trade** on DEXs
- **Stake inside XDC NFTs** to boost your yield up to 6%
- **Use as collateral** in DeFi protocols that support ERC-4626 vault tokens
- **Transfer** to any wallet — the recipient inherits the yield automatically

---

## Share-Based Pricing

Unlike V2 where 1 psXDC always equals 1 XDC, V3 uses a **share model**:

| Concept | How it works |
| --- | --- |
| **Deposit** | You deposit XDC, receive psXDC shares based on the current exchange rate |
| **Exchange rate** | `totalAssets / totalShares` — rises over time as rewards accrue |
| **Withdrawal** | You burn psXDC shares and receive XDC at the current rate |
| **Rewards** | No claiming needed — rewards increase the exchange rate for all holders equally |

**Example:** If the exchange rate is 1.05, burning 100 psXDC returns 105 XDC. The rate only goes up (assuming no validator losses).

This is the same model used by major DeFi protocols (Aave aTokens, Compound cTokens, Lido wstETH).

---

## Masternode Integration

V3 directly stakes XDC with masternodes via the on-chain XDC validator contract. This is what makes psXDC productive — your XDC is not sitting idle.

### How masternodes are managed

| Step | What happens |
| --- | --- |
| **Operator onboarding** | Admin adds KYC-verified masternode operators |
| **Auto-propose** | When excess liquid XDC exceeds the threshold, the contract automatically proposes a new masternode using round-robin operator selection |
| **Manual propose** | Proposer role can also trigger masternode proposals directly |
| **Rewards** | Validator rewards flow back into the contract, increasing the exchange rate |
| **Resignation** | Proposer/Admin can resign a masternode; after the network cooldown (~35 days), the XDC is withdrawn back |
| **Principal tracking** | The contract tracks exactly how much XDC is locked per operator, globally and individually |

### Liquidity buffer

The contract keeps a configurable percentage of total assets liquid (default 5%) to serve instant withdrawals. This means:

- Most XDC is working in masternodes earning yield
- A buffer remains in the contract for immediate redemptions
- If the buffer is depleted, the withdrawal queue handles the overflow automatically

---

## Governance & Roles

V3 separates responsibilities across five roles. No single key can make unilateral changes.

| Role | Responsibility |
| --- | --- |
| **Admin (Owner)** | Governance changes, operator management, KYC |
| **Operations Manager** | Day-to-day parameters: buffer %, scan limits, auto-propose config |
| **Risk Manager** | Report validator losses (capped per-report and per-day) |
| **Proposer** | Propose and resign masternodes |
| **Migration Manager** | Control the V2→V3 migration window |

### Time-locked changes

Every sensitive parameter change follows a three-step pattern:

1. **Schedule** the change → mandatory governance delay begins (default 1 day)
2. **Wait** — anyone can see the pending change on-chain
3. **Execute** the change after the delay expires — or **cancel** it at any time

This applies to: role changes, loss caps, governance delay itself, ownership transfer, and migration bridge configuration.

---

## Risk Controls

| Control | Detail |
| --- | --- |
| **No admin mint** | `mint()` is disabled — shares can only be created through genuine XDC deposits |
| **No admin withdraw** | There is no `ownerWithdraw()` — XDC only leaves through user redemptions or validator operations |
| **Loss caps** | `reportValidatorLoss()` is capped at 10% per report and 20% per day (configurable via time-lock) |
| **Principal separation** | Returning masternode principal is not counted as yield — prevents artificial exchange rate inflation |
| **Reentrancy protection** | All external state-changing functions are protected |
| **Reward threshold** | Reward inflows below 1,000 XDC are not immediately synced, preventing dust manipulation |
| **Zero slashing risk** | XDC Network does not implement slashing |

---

## Smart Contract Details

| Property | Detail |
| --- | --- |
| **Contract** | `PrimeStakedXDC_V3.sol` |
| **Standard** | ERC-4626 (OpenZeppelin) + AccessControl |
| **Proxy** | TransparentUpgradeableProxy |
| **Decimals offset** | 9 (extra share precision) |
| **Default masternode stake** | 10,000,000 XDC |
| **Default buffer** | 5% of total assets |
| **Default governance delay** | 1 day (min 1 minute, max 30 days) |
| **Audits** | QuillAudits + Nethermind Security |

---

## Migration from V2

Existing V2 psXDC holders migrate to V3 through a dedicated Migration Bridge contract:

1. Approve the bridge to spend your V2 psXDC
2. Call `migrate(amount, minSharesOut)` — the bridge burns your V2 tokens and mints V3 shares
3. Slippage protection (`minSharesOut`) prevents unfavorable exchange rates

The migration bridge has its own security controls: time-locked admin, daily withdrawal caps, and disabled direct role changes.

→ [Migration Guide](staking-guide/migration.md)
