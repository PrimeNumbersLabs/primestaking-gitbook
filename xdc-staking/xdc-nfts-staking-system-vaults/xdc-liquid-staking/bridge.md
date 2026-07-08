# Bridge psXDC to other chains

psXDC is native to the XDC Network, but you can bridge it to **Base**,
**Arbitrum**, and **BNB Chain** to use it in DeFi there — holding, providing
liquidity on DEXes, or (once listed) supplying it as collateral on money
markets. Bridging uses [LayerZero V2](https://layerzero.network), not a
third-party custodian.

{% hint style="success" %}
**Your stake keeps earning while it is bridged.** psXDC is non-rebasing: your
balance stays the same number, but each psXDC is redeemable for more XDC over
time as validator rewards compound into the vault share price. Bridging does not
pause or forfeit that yield — you capture it when you bridge back.
{% endhint %}

## How it works (lockbox)

psXDC has a single, canonical supply. When you bridge out, your real psXDC is
**locked** in the `PsxdcOFTAdapter` on XDC and an equal amount of psXDC is
**minted** on the destination chain. Bridging back **burns** the destination
psXDC and **unlocks** the original on XDC. Total psXDC across all chains never
changes.

```
XDC:         [ your psXDC ] --lock-->  PsxdcOFTAdapter (holds the real shares)
                                          |  LayerZero message (2 DVNs verify)
Base/Arb/BSC:                             v
                                       PsxdcOFT  --mint 1:1-->  [ your psXDC ]
```

## What you can and cannot do with bridged psXDC

| On XDC | On Base / Arbitrum / BSC |
| --- | --- |
| Stake / unstake through the vault | No — bridge back to XDC to unstake |
| Deposit into XDC NFTs for boosts | No — NFTs are XDC-only by design |
| Count toward the referral program | No — referral reads balances on XDC |
| Hold, transfer, DEX-LP | Hold, transfer, DEX-LP, use as collateral (once listed) |

To use any PrimeStaking feature (unstaking, NFT boosts, referrals), bridge your
psXDC back to XDC first.

## Seeing your yield on other chains

A wallet on Base/Arbitrum/BSC shows a fixed psXDC balance — the growth is in the
**redemption rate**, not the count. PrimeStaking publishes that rate to each
chain through the [psXDC rate oracle](psxdc-rate-oracle.md); the app's Bridge
page shows the current XDC value of your bridged balance.

## Security

- Every pathway is verified by **two independent DVNs** (LayerZero Labs and
  Nethermind) — a single compromised verifier cannot forge a bridge message.
- The adapter enforces a **per-destination rate limit**, bounding how fast the
  lockbox could be drained even in a worst-case destination-chain compromise,
  giving operators time to pause.
- Contracts are owned by PrimeStaking with two-step ownership transfer.

## Addresses

See [Deployed Contracts & Addresses](../contract-addresses.md) for the adapter
(XDC) and the psXDC OFT on each destination chain.
