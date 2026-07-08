# psXDC Rate Oracle (integrator guide)

For money markets and other integrators that want to price **bridged psXDC** on
Base, Arbitrum, or BNB Chain. psXDC is a yield-bearing vault share: its fair
value is `XDC price × psXDC/XDC exchange rate`. This oracle publishes that
exchange rate cross-chain from XDC so you can price psXDC exactly like a
wstETH/rETH rate provider.

## Interface

`PsxdcRateOracle` implements the Chainlink `AggregatorV3Interface`:

```solidity
function decimals() external view returns (uint8);        // 18
function latestRoundData() external view returns (
    uint80 roundId, int256 answer, uint256 startedAt, uint256 updatedAt, uint80 answeredInRound
);
function isStale() external view returns (bool);          // true past the staleness window
```

`answer` = XDC redeemable per **1e18** psXDC (18 decimals). Compose it with your
existing XDC/USD feed:

```
psXDC/USD = latestRoundData().answer * (XDC/USD) / 1e18
```

## Safety guarantees

The rate can only be written by the wired XDC publisher (LayerZero peer auth,
2 required DVNs). Each update is additionally guarded on the destination:

- **Monotonic** — the rate cannot decrease (share price only grows). A rare
  validator-loss writedown requires an explicit, one-shot owner override, so a
  spurious downward push is rejected by default.
- **Step-capped** — each update may rise by at most `maxStepBps` (default 1%),
  blunting a bad or manipulated push.
- **Staleness** — `updatedAt` plus `isStale()` let you reject data older than
  the configured window (default 2 days). PrimeStaking pushes the rate at least
  daily, and the push is permissionless so it cannot be censored.

## Recommended usage

Treat the oracle as the **exchange-rate** leg only; keep your own market
XDC/USD feed for the base asset. For collateral, apply your standard
loan-to-value and liquidation parameters to the composed psXDC/USD price, and
reject updates when `isStale()` is true.

## Addresses

Per-chain oracle addresses are in
[Deployed Contracts & Addresses](../contract-addresses.md). Reach out at
[admin@primenumbers.xyz](mailto:admin@primenumbers.xyz) for listing support.
