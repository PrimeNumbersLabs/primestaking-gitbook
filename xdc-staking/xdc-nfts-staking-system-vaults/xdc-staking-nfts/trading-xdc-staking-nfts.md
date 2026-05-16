---
hidden: true
---

# Trading XDC NFTs

XDC Staking NFTs are tradeable on [PrimePort.xyz](https://primeport.xyz). When you sell or transfer an NFT, the buyer inherits all staked psXDC v3 shares, weight, pending boost, and lock status.

---

## Selling

1. Go to [PrimePort.xyz](https://primeport.xyz) and connect your wallet.
2. Select the NFT you want to sell.
3. Set a price or start an auction.
4. Confirm the listing transaction.

Unlike the V2 model, V3 does **not** require you to first own 100% of the underlying psXDC — staked shares live inside the vault under the NFT's `tokenId` and travel with the NFT to the buyer automatically.

---

## Buying

1. Browse the XDC Staking NFTs collection on PrimePort.
2. Review the NFT's rarity, currently staked psXDC v3 shares, current weight, and price.
3. Click **Buy** and confirm the transaction.

After purchase, the NFT and its full staking state (`stakedShares`, `level`, `lockEnd`, `rewardIndex`, pending boost) transfer to your wallet.

{% hint style="warning" %}
Both V2 (legacy) and V3 NFTs may be listed. **V2 NFTs follow the legacy reward model** until they are migrated; **V3 NFTs immediately participate in the boost accumulator**. The collection address is the easiest way to tell them apart:

- V2 collection: `0x9D458330e458f11fd1cE7E44B3a66568af8076a0`
- V3 collection: [`0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E`](https://xdcscan.com/address/0xf3eB62F0Daf98ab65f0696630621A6ecECDB898E)

After buying a V2 NFT you can migrate it to V3 in a single transaction — same `tokenId`, same rarity, lock state preserved. See [Migrate XDC NFTs to V3](migrate-nfts-v2-to-v3.md).
{% endhint %}

---

## Transferring

1. Use the standard ERC-721 **Transfer** function on PrimePort or in the PrimeStaking app.
2. Confirm the transaction.

In V3 there is **no requirement to hold the underlying psXDC** before transferring — the shares are escrowed inside the vault, not in your wallet.

---

## Important Notes

- **Locked NFTs cannot be merged or burnt-and-redeemed** until `lockEnd` has passed, but they **can** still be transferred and traded.
- **Pending boost** travels with the NFT — when you transfer, the new owner can call `claim(tokenId)` to settle anything earned up to that point.
- **Fees** — PrimePort may charge a small marketplace fee.
- **Pricing** — NFT value depends on rarity, staked psXDC shares, level, lock status, and market demand.
