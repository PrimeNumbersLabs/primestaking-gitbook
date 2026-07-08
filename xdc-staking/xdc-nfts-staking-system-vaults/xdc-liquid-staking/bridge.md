# Bridge psXDC to other chains

psXDC is native to the XDC Network, but you can bridge it to **Base**,
**Arbitrum**, **BNB Chain**, and **HyperEVM** to put it to work in DeFi there —
lending it on [PrimeFi](https://primefi.xyz), providing liquidity, or simply
holding it on the chain where the rest of your portfolio lives. Bridging uses
[LayerZero V2](https://layerzero.network), not a third-party custodian.

See [Multichain psXDC: DeFi Opportunities](multichain-opportunities.md) for
what you can do with psXDC once it is bridged.

{% hint style="success" %}
**Your stake keeps earning while it is bridged.** psXDC is non-rebasing: your
balance stays the same number, but each psXDC is redeemable for more XDC over
time as validator rewards compound into the vault share price. Bridging does not
pause or forfeit that yield — you capture it when you bridge back.
{% endhint %}

## How to bridge (step by step)

The Bridge page is at
[primestaking.xyz/xdc-liquid-staking/bridge](https://primestaking.xyz/xdc-liquid-staking/bridge).

1. **Connect your wallet.** The page works from any of the five supported
   networks; it will ask your wallet to switch when needed.
2. **Pick the route.** Choose the source chain under *From* and the destination
   under *To* (routes always run through XDC: XDC ↔ Base/Arbitrum/BNB/HyperEVM).
   The circular arrow button flips the direction. Clicking a chain card in the
   balance strip below also selects it as the source.
3. **Enter the amount** (or press *Max*). The LayerZero fee quotes itself
   automatically and is shown in the route details, along with the estimated
   delivery time.
4. **Press Bridge.**
   - Bridging *out of XDC* for the first time asks for one extra approval
     transaction (the lockbox needs permission to hold your psXDC).
   - The bridge transaction itself carries the LayerZero fee as native gas
     (XDC when leaving XDC; ETH/BNB/HYPE when coming back).
5. **Track delivery.** A tracker card appears with a progress bar and a
   *LayerZero Scan* link. Delivery normally takes **2–5 minutes** (four
   independent verifiers must each confirm 20 source-chain blocks). The page
   detects arrival automatically and refreshes your balances.

### Fees

| What | Who charges it | Rough size |
| --- | --- | --- |
| LayerZero message fee | LayerZero (DVNs + executor) | a few XDC leaving XDC; small ETH/BNB/HYPE returning |
| Bridge fee / spread | **None** — PrimeStaking takes nothing | 0 |
| Amount received | 1:1 | what you send is what arrives |

## How it works (lockbox)

psXDC has a single, canonical supply. When you bridge out, your real psXDC is
**locked** in the `PsxdcOFTAdapter` on XDC and an equal amount of psXDC is
**minted** on the destination chain. Bridging back **burns** the destination
psXDC and **unlocks** the original on XDC. Total psXDC across all chains never
changes.

```
XDC:           [ your psXDC ] --lock-->  PsxdcOFTAdapter (holds the real shares)
                                            |  LayerZero message (4 DVNs verify)
Base/Arb/BSC/HyperEVM:                      v
                                         PsxdcOFT  --mint 1:1-->  [ your psXDC ]
```

The psXDC token contract is the **same address on every destination chain**:
`0x98D916F5773Ac0482b49856f2659d6c32114C4Ba`.

## What you can and cannot do with bridged psXDC

| On XDC | On Base / Arbitrum / BSC / HyperEVM |
| --- | --- |
| Stake / unstake through the vault | No — bridge back to XDC to unstake |
| Deposit into XDC NFTs for boosts | No — NFTs are XDC-only by design |
| Count toward the referral program | No — referral reads balances on XDC |
| Hold, transfer, DEX-LP | Hold, transfer, DEX-LP, **use as collateral on PrimeFi** (Base & HyperEVM) |

To use any PrimeStaking feature (unstaking, NFT boosts, referrals), bridge your
psXDC back to XDC first.

## Seeing your yield on other chains

A wallet on Base/Arbitrum/BSC/HyperEVM shows a fixed psXDC balance — the growth is in the
**redemption rate**, not the count. PrimeStaking publishes that rate to each
chain through the [psXDC rate oracle](psxdc-rate-oracle.md); the app's Bridge
page shows the current XDC value of your bridged balance.

## Security

- Every pathway is verified by **four independent DVNs** (Canary, LayerZero
  Labs, Horizen, and Nethermind) — a single compromised verifier cannot forge a
  bridge message.
- The adapter meters the **unlock path** with a per-source-chain rate limit.
  Because the lockbox only ever releases funds when psXDC is bridged *back*, the
  limit is applied there (keyed by the origin chain): even if one destination
  chain were fully compromised, it could unlock at most the configured burst
  immediately and a fixed rate thereafter, giving operators time to pause —
  while the other chains' pathways are unaffected.
- Total bridged psXDC across all chains always equals psXDC locked in the
  adapter — verified by an on-chain conservation test across multi-hop routes
  (XDC to Base to Arbitrum and back), including sub-1e12 dust round trips.
- Contracts use two-step ownership transfer (nominate + accept), so ownership
  cannot be fat-fingered to an address that cannot operate the lockbox.

## Troubleshooting

- **Transfer not arrived after 10+ minutes?** Open the *LayerZero Scan* link
  from the tracker card (or paste your transaction hash at
  [layerzeroscan.com](https://layerzeroscan.com)). Messages are never lost —
  delivery is retried automatically once verification completes.
- **"Insufficient balance" on the fee?** The LayerZero fee is paid in the
  source chain's gas token, on top of gas. Leaving XDC needs a few extra XDC;
  returning needs a small amount of ETH (Base/Arbitrum), BNB, or HYPE.
- **Wrong network in the wallet?** The page requests the switch automatically
  when you press Bridge; approve the prompt in your wallet.

## Addresses

See [Deployed Contracts & Addresses](../contract-addresses.md) for the adapter
(XDC) and the psXDC OFT + rate oracle on each destination chain.
