# Smart Contract Functions

**XDC Vaults Smart Contract Functions**

Core functions within the XDC Vaults smart contract, defining their purpose and usage.

***

#### **1. Mint Vault**

* **Description:** Users can mint a new XDC Vault directly from the staking platform.
* **Cost:** 100 XDC minting fee.
* **Functionality:**
  * The minted Vault acts as an on-chain asset where users can stake their XDC.
  * Each Vault is uniquely identified and tied to the user's wallet.
  * **Restriction: Users cannot mint a new Vault if they owe $pstXDC to the contract (i.e., if they have withdrawn $pstXDC from the wallet).**

***

#### **2. Stake**

* **Description:** Allows users to stake XDC tokens inside their Vault NFT to start earning staking rewards.
* **Functionality:**
  * Users deposit XDC into their Vault to begin accruing staking rewards.
  * Staking is non-destructive, meaning the NFT remains fully intact while the tokens remain staked.
  * **Restriction:** Users **cannot** stake more XDC if they owe $pstXDC to the contract, ensuring that all previous obligations are settled before additional staking.

***

#### **3. Withdraw XDC**

* **Description:** Users can withdraw their staked XDC tokens, removing their entire staking position.
* **Functionality:**
  * Users **must burn** their Vault NFT and the corresponding $pstXDC when withdrawing their XDC.
  * The withdrawal process removes the fully staked position, ensuring that no duplicate claims on staking rewards occur.
  * Withdrawn XDC is transferred back to the user’s wallet.

***

#### **4. Transfer**

* **Description:** Allows users to move their NFT between wallets.
* **Functionality:**
  * Vault ownership is updated on-chain when transferred to another user.
  * The new owner gains full control over the Vault and its staking capabilities.

***

#### **5. Claim XDC Rewards**

* **Description:** Users can claim their monthly $pstXDC rewards.
* **Functionality:**
  * Rewards are credited monthly and stored within the NFT.
  * **Requirement:** Users must hold **100% of the $pstXDC** linked to the Vault at the moment of claiming.
  * Partial $pstXDC holders are ineligible for rewards, ensuring fair distribution.

***

#### **6. Sell**

* **Description:** Users can list or auction their XDC Vault NFT on the **PrimePort marketplace**.
* **Functionality:**
  * Sellers can define their price and list their Vault for bidding or direct purchase.
  * The new buyer inherits the Vault’s staked XDC and any applicable rewards.

***

#### **7. Transfer (Sale or Gifting)**

* **Description:** Users can sell or transfer an XDC Staking NFT.
* **Functionality:**
  * **Requirement:** The user must hold all $pstXDC tokens tied to the NFT.
  * When transferred, all staking rewards and ownership rights shift to the recipient.

***

#### **Security & Ownership Rules**

* Users must maintain 100% of their $pstXDC to claim staking rewards or execute transfers.
* Users cannot mint new Vaults or stake additional XDC if they owe $pstXDC to the contract.
* To withdraw staked XDC, the Vault NFT and its corresponding $pstXDC must be burned, fully removing the staking position.
* Transactions are recorded on-chain, ensuring full transparency and ownership integrity.
