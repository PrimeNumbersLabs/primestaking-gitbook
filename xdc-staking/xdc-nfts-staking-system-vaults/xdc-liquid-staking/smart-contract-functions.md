---
hidden: true
---

# Smart Contract Reference

Technical reference for the XDC Liquid Staking smart contract functions.

---

## Core Functions

### Mint Vault

Creates a new XDC staking vault (represented as an on-chain NFT).

| Parameter | Detail |
| --- | --- |
| **Cost** | 100 XDC minting fee |
| **Restriction** | Cannot mint if the user owes psXDC to the contract |

The vault is tied to the user's wallet and serves as the container for staked XDC.

---

### Stake

Deposits XDC tokens into a vault NFT to begin earning rewards.

- Non-destructive: the NFT remains intact while tokens are staked.
- **Restriction:** Cannot stake additional XDC if the user owes psXDC to the contract.

---

### Withdraw XDC

Removes the entire staking position from a vault.

- Requires burning both the vault NFT and corresponding psXDC.
- Withdrawn XDC is returned to the user's wallet after the validator queue processes the request (~31 days average).

---

### Transfer

Moves a vault NFT to a different wallet. The new owner gains full control over the vault and its staking capabilities.

---

### Claim Rewards

Claims accrued psXDC rewards.

- Rewards accrue continuously and are stored within the contract.
- **Requirement:** The user must hold 100% of the psXDC linked to the vault at claim time.

---

### Sell

Lists or auctions the vault NFT on the [PrimePort marketplace](https://primeport.xyz). The buyer inherits the vault's staked XDC, rewards, and all associated rights.

---

## Security Rules

- Users must hold 100% of their vault's psXDC to claim rewards or execute transfers.
- Minting and additional staking are blocked if the user owes psXDC to the contract.
- Withdrawal requires burning both the vault NFT and its psXDC, fully closing the position.
- All transactions are recorded on-chain for full transparency.
