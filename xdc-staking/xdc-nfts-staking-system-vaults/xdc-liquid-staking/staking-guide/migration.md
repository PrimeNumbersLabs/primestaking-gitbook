# Migrate V2 psXDC → V3.1

{% hint style="success" %}
**Held psXDC V3 before July 2026? You're already done.** When the protocol redeployed the vault as V3.1, every V3 holder's balance was mirrored 1:1 onto the new contract via a snapshot airdrop, including balances inside XDC NFTs, DEX liquidity pools, and lending markets, which were credited to their underlying owners. No transaction, claim, or approval was needed. The app already shows your V3.1 balance.
{% endhint %}

This page is for users who still hold **V2 psXDC** (the original 1:1 token at [`0x9B8e…65A6`](https://xdcscan.com/address/0x9B8e12b0BAC165B86967E771d98B520Ec3F665A6)). Your V2 balance does **not** automatically appear in V3.1. You migrate by sending V2 tokens through the dedicated migration bridge, which burns the V2 and mints V3.1 shares in one transaction.

{% hint style="info" %}
Migration is optional at any individual point in time, but V3.1 is the only version with self-service withdrawals, share-price reward accrual, and NFT-boost compatibility. See [Legacy (V2, historical)](../../../../legacy/README.md).
{% endhint %}

---

## What the bridge does

```
   User                  v2 → V3.1 Migration Bridge           psXDC V3.1 vault
    │                              │                                │
    │  approve(bridge, amount)     │                                │
    │ ───────────────────────────►│                                │
    │                              │                                │
    │  migrate(amount, minSharesOut)                                │
    │ ───────────────────────────►│                                │
    │                              │  burns V2 psXDC                │
    │                              │  vault.migrate(receiver, …) ──►│
    │                              │                                │  mints V3.1 shares
    │ ◄─────────────────── V3.1 shares (≥ minSharesOut) ◄───────────│
```

- **Burns your V2 psXDC** so each V2 token can only be migrated once.
- **Mints V3.1 shares** to your address using the vault's `migrate(...)` function, which is restricted to the configured bridge.
- **Slippage protection** via `minSharesOut` so the migration reverts if the exchange rate changed unfavourably between the time you signed and the time the transaction settled.
- **Atomic**: if any step fails (slippage exceeded, etc.) the whole transaction reverts and you keep your V2 tokens.

The live bridge is [`0x6c57075c7A157113D369109B78738A798d41373C`](https://xdcscan.com/address/0x6c57075c7A157113D369109B78738A798d41373C). The pre-V3.1 bridge (`0x2927…5dd2`) is retired; the app never routes to it.

---

## Steps

1. Go to the migration page at [primestaking.xyz/xdc-liquid-staking/migration](https://primestaking.xyz/xdc-liquid-staking/migration). (The app also shows a migration banner on the main staking dashboard.)
2. Enter the amount of V2 psXDC you want to migrate. The app previews the V3.1 shares you will receive and applies a default slippage tolerance of **0.5%**, which you can adjust.
3. Click **Approve** to allow the bridge to spend that amount of V2 psXDC. Confirm in your wallet.
4. Click **Migrate**. The app calls `migrate(amount, minSharesOut)` on the bridge. Confirm the transaction.
5. After the transaction settles, your V3.1 share balance increases by at least `minSharesOut`, your V2 balance decreases by `amount`, and you can use the shares like any newly-staked position (transfer, redeem instantly or via the queue, deposit into a V3 NFT).

---

## Exchange rate and slippage

The V3.1 vault uses share-based pricing (`totalAssets / totalShares`), so each migrated V2 psXDC mints a share count proportional to the current exchange rate:

- If the exchange rate is `1.00`, migrating `1,000` V2 psXDC mints `1,000` V3.1 shares.
- If the exchange rate has appreciated to `1.05`, the same migration mints `~952.38` shares (each share is now worth more XDC).

The default 0.5% slippage tolerance covers small mid-transaction movement. If you set `minSharesOut` too high (e.g. expecting yesterday's rate) and the rate has since grown, the migration will revert. Adjust slippage and retry.

---

## What happened to old V3 psXDC?

The pre-July-2026 V3 token ([`0x98D9…C4Ba`](https://xdcscan.com/address/0x98D916F5773Ac0482b49856f2659d6c32114C4Ba)) was superseded in full:

- A **snapshot** captured every holder's balance, with special handling so nothing was missed: psXDC held inside XDC NFTs stayed with the NFT vault, DEX LP positions were unwound pro-rata to the liquidity providers, lending-market deposits were credited to depositors, and open limit orders were returned to their makers.
- The **`V31AirdropDistributor`** minted those exact balances on V3.1 in the vault's final state, then was permanently finalized.
- The old V3 contract's bridge was cut to a dead address; old V3 tokens have no remaining function and cannot be migrated or redeemed. Your value lives in V3.1.

If you believe a balance was missed, contact [admin@primenumbers.xyz](mailto:admin@primenumbers.xyz); the full snapshot and airdrop are auditable on-chain via the distributor contract.

---

## What happens to V2 after you migrate

- The V2 contract continues to exist for users who have not migrated.
- Your migrated V2 tokens are **burned** and cannot be re-issued.
- V2-side actions only apply to whatever V2 balance you have left.

→ [Withdrawals: Instant vs Queued (V3.1)](withdrawals-instant-vs-queued.md) → [V2 vs V3: What Changed](../v2-vs-v3.md) → [Historical pstXDC → psXDC migration (legacy)](../../../../legacy/pstxdc-migration.md)
