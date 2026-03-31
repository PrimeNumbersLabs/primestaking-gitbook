# Request Withdrawal

To convert psXDC back to XDC, you submit a withdrawal request. The request is processed via the validator queue.

---

### Steps

1. Go to [primestaking.xyz/xdc-liquid-staking](https://primestaking.xyz/xdc-liquid-staking/overview).
2. Open the **Withdraw** section.
3. Enter the amount of psXDC you want to burn.
4. Click **Request Withdrawal** and confirm the transaction.

---

### Processing Time

Withdrawal requests are processed via the validator queue. The average processing time is **~35 days** under normal network conditions.

This delay comes from the XDC Network's fixed `candidateWithdrawDelay` of 1,296,000 blocks. While the theoretical block time is 2.00s (which would equal 30 days), real-world block times average ~2.33s, resulting in ~35 days. During periods of network congestion, it can take longer.

Once your withdrawal is ready, you can execute it from the **My Positions** page to receive XDC back in your wallet.

<figure><img src="../../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>
