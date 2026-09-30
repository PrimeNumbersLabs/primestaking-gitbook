# Multichain psXDC: DeFi opportunities

Bridging psXDC is not just about moving tokens — it turns your staked XDC into
**productive collateral** on other ecosystems while it keeps earning XDC
staking rewards underneath. This page lists what is live today and how the
yield stacks.

{% hint style="info" %}
psXDC is the same asset everywhere: one canonical supply, locked 1:1 on XDC,
minted at the same address `0x98D916F5773Ac0482b49856f2659d6c32114C4Ba` on
Base, Arbitrum, BNB Chain, and HyperEVM. See
[Bridge psXDC](bridge.md) for how to move it.
{% endhint %}

## Live today: lend and borrow on PrimeFi (Base & HyperEVM)

psXDC is listed as **collateral** on [PrimeFi](https://primefi.xyz), the
Aave-style money market, on both **Base** and **HyperEVM**.

What that unlocks:

- **Supply psXDC, earn supply APY** on top of the XDC staking yield already
  accruing inside the token. Two yields, one asset.
- **Borrow against your stake without unstaking.** Supply psXDC as collateral
  and borrow stablecoins or other listed assets — keep your XDC staking
  exposure and rewards while freeing liquidity for anything else.
- **No lock-up interaction.** Withdrawing supplied psXDC from PrimeFi is
  instant (subject to pool utilisation); your underlying stake on XDC is never
  touched.

How psXDC is priced there: PrimeFi consumes an XDC/USD feed and treats psXDC
conservatively at the XDC price. Because the psXDC redemption rate only grows,
your collateral is if anything slightly *under*-valued — a safety margin, not a
risk. The on-chain [psXDC rate oracle](psxdc-rate-oracle.md) is available for
integrators who want to credit the full redemption value.

{% hint style="warning" %}
**Borrowing carries liquidation risk.** Collateral parameters (LTV,
liquidation threshold and bonus) are set by PrimeFi and shown live in the
PrimeFi app — always size loans against those numbers, not this page. Start
conservative; borrowing stablecoins against a yield-bearing asset is a
strategy, not free money.
{% endhint %}

### The yield stack, concretely

| Layer | Source | Where it shows up |
| --- | --- | --- |
| ~4.5% APY XDC staking | validator rewards on XDC | psXDC redemption rate (steps up when the network pays monthly masternode rewards) |
| Supply APY | PrimeFi lenders' market | claimable on PrimeFi |
| Borrowed capital | whatever you deploy it into | your strategy |

Example: bridge psXDC to Base → supply on PrimeFi → borrow a stablecoin at a
conservative loan-to-value → deploy the stablecoin (LP, yield, or simply hold
as dry powder). Your XDC stake keeps compounding the entire time, and you can
unwind any moment: repay → withdraw → bridge back → unstake or sell on
[Spot](../dex-liquidity.md).

## Also available on every destination chain

- **Hold & transfer** — psXDC is a plain ERC-20 on Base, Arbitrum, BNB Chain,
  and HyperEVM. Self-custody it, move it between your own wallets, or pay
  another psXDC holder, all with the yield still accruing.
- **DEX liquidity** — anyone can seed a psXDC pair on a destination-chain DEX.
  Pricing helpers can read the [psXDC rate oracle](psxdc-rate-oracle.md)
  (same address on every chain:
  `0x2927630dfDd66433DbA9370b316EF5a8408d5dD2`).

## For protocols who want to integrate psXDC

Money markets, DEXes, and vaults on any of the four destination chains can list
psXDC with two addresses and one page of reading:

- Token (all chains): `0x98D916F5773Ac0482b49856f2659d6c32114C4Ba`
- Rate oracle, Chainlink `AggregatorV3Interface`, answer = XDC per psXDC,
  18 decimals (all chains): `0x2927630dfDd66433DbA9370b316EF5a8408d5dD2`
- Integrator guide: [psXDC Rate Oracle](psxdc-rate-oracle.md)

The oracle is monotonic, step-capped, and refreshed daily — designed
specifically so lending protocols can treat psXDC like a wstETH-class
liquid-staking collateral.

## Where the yield ultimately comes from

Nothing on this page changes how rewards are generated: XDC validator rewards
flow into the vault on XDC and raise the psXDC share price
([how rewards work](xdc-staking-rewards.md)). Bridging, lending, and LP-ing
are layers *on top* of that base yield — every bridged psXDC remains a claim
on the same appreciating vault share, redeemable the moment it returns to XDC.
