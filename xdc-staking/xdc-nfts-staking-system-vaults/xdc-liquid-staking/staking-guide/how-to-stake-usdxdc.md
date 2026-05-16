# How to Stake XDC

### Prerequisites

- An XDC-compatible wallet (e.g., MetaMask configured for XDC Network).
- XDC tokens in your wallet.

---

### Step 1 - Connect Your Wallet

Go to [primestaking.xyz](https://primestaking.xyz) and click **Connect Wallet**. Approve the connection in your wallet.

<figure><img src="../../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

---

### Step 2 - Navigate to XDC Liquid Staking

Open [primestaking.xyz/xdc-liquid-staking](https://primestaking.xyz/xdc-liquid-staking/overview) and enter the amount of XDC you want to stake. The app previews how many psXDC shares you will receive at the current exchange rate — this is **not** a fixed 1:1; once the vault has accrued rewards, each share is worth more than 1 XDC.

<figure><img src="../../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

---

### Step 3 - Confirm the Transaction

Review the previewed share amount and click **Stake**. The app calls `stake()` on the V3 vault with your XDC as `msg.value`. Confirm the on-chain transaction in your wallet.

---

### Step 4 - Receive psXDC Shares

Once confirmed, psXDC shares are minted to your wallet at the live exchange rate. From this moment your shares are appreciating — rewards accrue automatically through the share price, with no claim button to press.

You can now hold psXDC, transfer it, trade it on a DEX, use it as DeFi collateral, or deposit it into an [XDC NFT](../../xdc-staking-nfts/) for the additional boost slice.

---

### Adding psXDC to Your Wallet

If psXDC doesn't appear in your wallet automatically, click the **Add psXDC Token** button in the app.

<figure><img src="../../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Alternatively, add it manually as a custom token:

**Contract address (V3):** [`0x98D916F5773Ac0482b49856f2659d6c32114C4Ba`](https://xdcscan.com/address/0x98D916F5773Ac0482b49856f2659d6c32114C4Ba)

> Note: the **V2** psXDC token address (`0x9B8e12b0BAC165B86967E771d98B520Ec3F665A6`) is different. If you stake fresh XDC you will receive V3 shares only. If you still hold V2 tokens, see [Migrate V2 psXDC → V3](migration.md).

![](<../../../../.gitbook/assets/image (2) (1).png>)

![](<../../../../.gitbook/assets/image (3).png>)

![](<../../../../.gitbook/assets/image (4).png>)

<div align="left"><figure><img src="../../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure></div>
