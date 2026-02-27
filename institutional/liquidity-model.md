# Liquidity Model

PrimeStaking's liquidity model ensures that psXDC remains a usable, tradeable asset while staked XDC generates validator rewards.

---

## psXDC Liquidity

### Token Properties

| Property | Detail |
| --- | --- |
| **Name** | psXDC (Prime Staked XDC) |
| **Mint ratio** | 1 XDC = 1 psXDC (always) |
| **Network** | XDC Network |
| **Transferable** | Yes - standard XRC-20 token |
| **Earns rewards** | Yes - holding psXDC accrues staking rewards |
| **Redeemable** | Yes - burn psXDC to withdraw XDC |

### Minting & Redemption

- **Minting:** Users deposit XDC and receive psXDC at a 1:1 ratio, instantly
- **Redemption:** Users burn psXDC to initiate a withdrawal; XDC is returned after validator queue processing (~31 days average)
- **No slippage on mint/burn:** The ratio is protocol-enforced at 1:1

### Market Liquidity

psXDC is paired with XDC on the **XSWAP DEX**, providing a secondary market for users who want instant exit without waiting for the validator queue.

| DEX | Pair | Link |
| --- | --- | --- |
| XSWAP | psXDC / XDC | [Pool](https://info.xspswap.finance/#/pools/0xc4a0b4ce176c623a281bc565bfd35eab4fd7050a) |

Partners should note that DEX liquidity depth determines how large a position can be exited instantly without price impact. The protocol redemption path (burn psXDC → wait ~31 days) is always available regardless of DEX liquidity conditions, and guarantees 1:1 redemption.

---

## Withdrawal Queue

### How It Works

1. User submits a withdrawal request (burns psXDC)
2. The protocol initiates validator unstaking
3. XDC is returned after the XDC Network validator queue processes the request
4. Average processing time: **~31 days**

The ~31 day timeline is determined by the XDC Network's validator unstaking mechanism and is standard across all XDC staking implementations - not specific to PrimeStaking.

### Implications for Partners

| Consideration | Detail |
| --- | --- |
| **Not instant** | Withdrawals are subject to network-level validator unstaking delays |
| **Predictable** | ~31 day average; transparent queue position |
| **No partial fills** | Full withdrawal amount is returned once processed |
| **Instant alternative** | Users can swap psXDC on XSWAP DEX for immediate exit (subject to pool depth) |
| **User communication** | Partners should clearly communicate both withdrawal paths to users |

---

## Liquidity Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- |
| **psXDC depegs from XDC** | Low | Medium | DEX liquidity pool, arbitrage incentives, 1:1 protocol redemption |
| **DEX liquidity dries up** | Low | Low | Protocol redemption always available (with queue delay) |
| **Mass withdrawal event** | Low | Medium | Validator queue processes requests sequentially; no protocol insolvency risk |

---

## Key Metrics for Partners

All metrics below are verifiable on-chain:

| Metric | Description | Where to Check |
| --- | --- | --- |
| **Total psXDC supply** | Total staked XDC in the protocol | [XDCScan](https://xdcscan.com/token/xdc59e51346a6e1d05a0017c0e5f0501dcbba41bca1) |
| **DEX liquidity depth** | Available liquidity in psXDC/XDC pool | [XSWAP Pool](https://info.xspswap.finance/#/pools/0xc4a0b4ce176c623a281bc565bfd35eab4fd7050a) |
| **Withdrawal queue length** | Number of pending withdrawal requests | On-chain |
| **Average processing time** | Historical average for withdrawal completion | On-chain |
| **Peg ratio** | Current market price of psXDC vs XDC | XSWAP price feed |
