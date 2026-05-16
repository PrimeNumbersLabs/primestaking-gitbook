# Liquidity Model

PrimeStaking V3 separates **protocol redemption** (burning shares for XDC against the vault) from **market price** (psXDC on a DEX). Both stay healthy for partner integrations, but they behave differently and the difference is important.

***

## psXDC V3 Token Properties

| Property | Detail |
| --- | --- |
| **Name** | psXDC (Prime Staked XDC) |
| **Standard** | ERC-4626 vault share |
| **Pricing** | Exchange rate `totalAssets / totalShares` — grows over time as validator rewards accrue. **Not a fixed 1:1.** |
| **Network** | XDC Network (chain ID `50`) |
| **Address** | [`0x98D916F5773Ac0482b49856f2659d6c32114C4Ba`](https://xdcscan.com/address/0x98D916F5773Ac0482b49856f2659d6c32114C4Ba) |
| **Transferable** | Yes — standard XRC-20 token |
| **Yield mechanism** | Share-price appreciation; rewards are already inside the share — no manual `claim` |
| **Redeemable against the protocol** | Yes — `withdraw` / `redeem` (instant when buffer suffices) or `redeemWithQueue` (instant or queued) |
| **Market-tradeable** | Yes — XSWAP and other DEXs |

***

## Two distinct exit paths

### 1. Protocol redemption (NAV-based)

When you burn psXDC against the vault, you receive XDC at the **current exchange rate**, not a fixed 1:1. The amount is `convertToAssets(shares)`.

- **Instant** when the vault's liquid buffer (default 5% of total assets, configurable via `setBufferBps`) covers your request — settled in the same transaction.
- **Queued** when the buffer is insufficient — escrows the shares, adds you to the FIFO queue. Processed as new deposits / reward inflows / masternode resignations replenish liquidity.
- **No partial fills** — each request settles in full when its turn comes.
- **Free cancellation** — you can `cancelQueuedWithdrawal` any time before settlement and get your shares back unchanged.

For very large redemptions where the vault doesn't have a sufficient buffer + recent rewards, the upper bound on settlement is the XDC Network's `candidateWithdrawDelay` — about 35 days under typical real-world block times. This delay is a network-level property of XDC's masternode unstaking flow, not a PrimeStaking-specific gate.

→ [Withdrawals: Instant vs Queued](../xdc-staking/xdc-nfts-staking-system-vaults/xdc-liquid-staking/staking-guide/withdrawals-instant-vs-queued.md)

### 2. Market exit (DEX swap)

Holders can swap psXDC for XDC on a DEX (e.g. [XSWAP](https://info.xspswap.finance/#/pools/0xc4a0b4ce176c623a281bc565bfd35eab4fd7050a)). Market price is set by the AMM and is influenced by:

- The vault's current NAV (`totalAssets / totalShares`) — the long-run anchor.
- Pool depth and recent volume.
- Demand for immediate liquidity vs willingness to wait for protocol redemption.

The market price can sit **at, above, or below NAV** depending on these factors. Arbitrageurs can close large gaps using the protocol's instant redemption path when liquidity is available, but small NAV/market deltas are normal.

| Concept | NAV (protocol) | Market price (DEX) |
| --- | --- | --- |
| **Reference** | `totalAssets / totalShares` | AMM pool ratio |
| **Updates** | When rewards or losses are reflected on-chain | Continuously via trades |
| **Slippage** | None for amounts within the buffer; queue otherwise | Subject to pool depth |
| **Settlement** | Instant or queued by the vault | Atomic at trade execution |

***

## Implications for Partners

| Consideration | Detail |
| --- | --- |
| **Two value references** | A user's psXDC balance can be valued either at NAV (`convertToAssets`) or at DEX market price. Use NAV for accounting and partner reporting; DEX price for synchronous trading flows. |
| **Self-service withdrawals** | Partners do not need to operate any approval flow; users can redeem on their own through `redeemWithQueue`. |
| **Buffer planning** | If you expect bursty user withdrawal patterns, coordinate with PrimeStaking on buffer sizing (`setBufferBps`) — this affects how often partner users hit the queue. |
| **Queue UX** | Partners surfacing psXDC redemption should distinguish "complete" from "queued, self-claim later" outcomes — the contract makes this explicit through events. |
| **DEX integration** | psXDC pools provide an instant exit, but the protocol's NAV is the canonical value and is what should drive any institutional reporting. |

***

## Liquidity Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- |
| psXDC trades at a discount to NAV on a DEX | Medium | Low | Arbitrageurs can buy on the DEX and redeem at NAV when the queue allows; the protocol cannot guarantee zero discount in all conditions. |
| Mass redemption event | Low | Medium | FIFO queue ensures fair sequencing; the protocol cannot become insolvent because each redemption settles at the live share rate against on-chain assets. |
| Buffer is too small for normal partner traffic | Low | Medium | `setBufferBps` is governed by `OPERATIONS_MANAGER_ROLE` and can be tuned through the standard ops process. |
| Failed receiver payout | Very low | Low | Deferred into `pendingQueuedAssets` and claimable via `claimQueuedAssets`. No XDC is lost. |

***

## Key Metrics for Partners

All metrics below are verifiable on-chain via the [staking-v3-indexer](https://github.com/PrimeNumbersLabs/staking-v3-indexer) or direct calls to `PrimeStakedXDC_V3`.

| Metric | Description | Source |
| --- | --- | --- |
| **psXDC total supply** | Outstanding V3 shares | `PrimeStakedXDC_V3.totalSupply()` |
| **Total assets** | XDC tracked by the vault (buffer + delegated) | `PrimeStakedXDC_V3.totalAssets()` |
| **Exchange rate** | `totalAssets / totalShares` (or `convertToAssets(1e18)`) | Vault view |
| **Buffer ratio** | Current liquid XDC vs target buffer | Computed from `getBalance` + `bufferBps` |
| **Queue depth** | Outstanding `WithdrawalQueued` requests | Subgraph |
| **Per-operator principal** | XDC delegated per masternode operator | `outstandingValidatorPrincipalByOperator(operator)` |
| **DEX market price** | psXDC/XDC pool quote | XSWAP price feed |

Partners evaluating large position sizes should review the **buffer** and **queue depth** in parallel with DEX pool depth, and reach out to discuss sizing.

→ [Reward Mechanics](reward-mechanics.md) → [Risk & Compliance](risk-and-compliance.md) → [Deployed Contracts & Addresses](../xdc-staking/xdc-nfts-staking-system-vaults/contract-addresses.md)
