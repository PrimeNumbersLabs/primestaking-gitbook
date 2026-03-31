# Liquidity Model

PrimeStaking's liquidity model ensures that psXDC remains a usable, tradeable asset while staked XDC generates validator rewards.

***

## psXDC Liquidity

### Token Properties

| Property          | Detail                                      |
| ----------------- | ------------------------------------------- |
| **Name**          | psXDC (Prime Staked XDC)                    |
| **Mint ratio**    | 1 XDC = 1 psXDC (always)                    |
| **Network**       | XDC Network                                 |
| **Transferable**  | Yes - standard XRC-20 token                 |
| **Earns rewards** | Yes - holding psXDC accrues staking rewards |
| **Redeemable**    | Yes - burn psXDC to withdraw XDC            |

### Minting & Redemption

* **Minting:** Users deposit XDC and receive psXDC at a 1:1 ratio, instantly
* **Redemption:** Users burn psXDC to initiate a withdrawal; XDC is returned after validator queue processing (\~35 days average)
* **No slippage on mint/burn:** The ratio is protocol-enforced at 1:1

***

## Withdrawal Queue

### How It Works

1. User submits a withdrawal request (burns psXDC)
2. The protocol initiates validator unstaking
3. XDC is returned after the XDC Network validator queue processes the request
4. Average processing time: **\~35 days** under normal network conditions

The withdrawal delay is determined by the XDC Network's `candidateWithdrawDelay` parameter, which is fixed at **1,296,000 blocks**. At the theoretical 2.00-second block time, this equals 30 days. However, real-world block times consistently exceed the theoretical target:

| Period | Avg block time | Effective withdrawal delay |
| ------ | -------------- | ------------------------- |
| Theoretical | 2.00s | 30 days |
| 12-month average | \~2.33s | \~35 days |
| 90-day periods (with congestion) | \~2.83s | \~42 days |

Under normal conditions, users should expect **\~35 days**. During network congestion events, the effective delay can extend further. This is standard across all XDC staking implementations - not specific to PrimeStaking.

For context, Ethereum liquid staking protocols (Lido, Rocket Pool) also implement withdrawal queues that can range from days to weeks depending on network conditions.

### Implications for Partners

| Consideration           | Detail                                                                       |
| ----------------------- | ---------------------------------------------------------------------------- |
| **Not instant**         | Withdrawals are subject to network-level validator unstaking delays          |
| **Predictable**         | \~35 day average based on real-world block times; transparent queue position  |
| **No partial fills**    | Full withdrawal amount is returned once processed                            |
| **Instant alternative** | Users can swap psXDC on XSWAP DEX for immediate exit (subject to pool depth) |
| **User communication**  | Partners should clearly communicate both withdrawal paths to users           |

***

## Liquidity Risk Assessment

| Risk                      | Likelihood | Impact | Mitigation                                                                   |
| ------------------------- | ---------- | ------ | ---------------------------------------------------------------------------- |
| **psXDC depegs from XDC** | Low        | Medium | DEX liquidity pool, arbitrage incentives, 1:1 protocol redemption            |
| **Mass withdrawal event** | Low        | Medium | Validator queue processes requests sequentially; no protocol insolvency risk |

***

## Key Metrics for Partners

All metrics below are verifiable on-chain:

| Metric                      | Description                                  | Where to Check                                                                   |
| --------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------- |
| **Total psXDC supply**      | Total staked XDC in the protocol             | [XDCScan](https://xdcscan.com/token/xdc59e51346a6e1d05a0017c0e5f0501dcbba41bca1) |
| **Withdrawal queue length** | Number of pending withdrawal requests        | On-chain                                                                         |
| **Average processing time** | Historical average for withdrawal completion | On-chain                                                                         |
| **Peg ratio**               | Current market price of psXDC vs XDC         | XSWAP price feed                                                                 |

Current pool depth and volume data is available directly on XSWAP. Partners evaluating large position sizes should review pool depth before integration and contact us to discuss liquidity arrangements for institutional volumes.
