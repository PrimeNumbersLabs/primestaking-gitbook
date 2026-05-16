# Migrate V2 psXDC → V3

V3 is deployed on a brand-new vault contract ([`PrimeStakedXDC_V3`](../../contract-addresses.md)). Your V2 psXDC balance does **not** automatically appear in V3 — you migrate by sending V2 tokens through the dedicated migration bridge, which burns the V2 and mints V3 shares in one transaction.

{% hint style="info" %}
Migration is optional at any individual point in time, but V3 is the only version with self-service withdrawals, share-price reward accrual, and NFT-boost compatibility. V2 contracts remain live for users who haven't migrated yet — see [Legacy (V2 — historical)](../../../../legacy/README.md).
{% endhint %}

---

## What the bridge does

```
   User                    V3 Migration Bridge                psXDC V3 vault
    │                              │                                │
    │  approve(bridge, amount)     │                                │
    │ ───────────────────────────►│                                │
    │                              │                                │
    │  migrate(amount, minSharesOut)                                │
    │ ───────────────────────────►│                                │
    │                              │  burns V2 psXDC                │
    │                              │  vault.migrate(receiver, …) ──►│
    │                              │                                │  mints V3 shares
    │ ◄──────────────────── V3 shares (≥ minSharesOut) ◄────────────│
```

- **Burns your V2 psXDC** so each V2 token can only be migrated once.
- **Mints V3 shares** to your address using the V3 vault's `migrate(...)` function, which is restricted to the configured bridge while the migration window is open.
- **Slippage protection** via `minSharesOut` so the migration reverts if the exchange rate changed unfavourably between the time you signed and the time the transaction settled.
- **Atomic** — if any step fails (window closed, slippage exceeded, etc.) the whole transaction reverts and you keep your V2 tokens.

---

## Steps

1. Go to the migration page at [primestaking.xyz/xdc-liquid-staking/migration](https://primestaking.xyz/xdc-liquid-staking/migration). (The app also shows a migration banner on the main staking dashboard.)
2. Enter the amount of V2 psXDC you want to migrate. The app previews the V3 shares you will receive and applies a default slippage tolerance of **0.5%**, which you can adjust.
3. Click **Approve** to allow the bridge to spend that amount of V2 psXDC. Confirm in your wallet.
4. Click **Migrate**. The app calls `migrate(amount, minSharesOut)` on the bridge. Confirm the transaction.
5. After the transaction settles, your V3 share balance increases by at least `minSharesOut`, your V2 balance decreases by `amount`, and you can use the V3 shares like any newly-staked position (transfer, redeem instantly or via the queue, deposit into a V3 NFT).

---

## Exchange rate and slippage

The V3 vault uses share-based pricing (`totalAssets / totalShares`), so each migrated V2 psXDC mints a share count proportional to the current exchange rate:

- If the V3 exchange rate is `1.00`, migrating `1,000` V2 psXDC mints `1,000` V3 shares.
- If the V3 exchange rate has appreciated to `1.05`, the same migration mints `~952.38` V3 shares (each share is now worth more XDC).

The default 0.5% slippage tolerance covers small mid-transaction movement. If you set `minSharesOut` too high (e.g. expecting yesterday's rate) and the rate has since grown, the migration will revert — adjust slippage and retry.

---

## Migration window

The bridge can only mint V3 shares while the `MIGRATION_MANAGER_ROLE` has set `setMigrationInProgress(true)` on the vault. If the window is currently closed:

- The migration page in the app shows a "Migration not active" banner.
- The transaction would revert with `MigrationNotInProgress` — the app blocks the button so you never sign a doomed tx.

PrimeStaking maintains an open migration window for as long as there are V2 holders. If the window has been temporarily paused (e.g. for an operational change), it will be reopened.

---

## What happens to V2 after you migrate

- The V2 contract continues to operate for users who have not migrated.
- Your migrated V2 tokens are **burned** — they cannot be re-issued.
- V2-side actions (withdraw, claim) only apply to whatever V2 balance you have left.

→ [Withdrawals: Instant vs Queued (V3)](withdrawals-instant-vs-queued.md) → [V2 vs V3 — What Changed](../v2-vs-v3.md) → [Historical pstXDC → psXDC migration (legacy)](../../../../legacy/pstxdc-migration.md)
