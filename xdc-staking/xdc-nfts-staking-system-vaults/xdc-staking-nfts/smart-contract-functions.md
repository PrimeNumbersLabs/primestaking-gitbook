# Smart Contract Functions

Core functions within the XDC Staking NFTs smart contract, defining their purpose and usage.

***

#### **1. Stake**

* **Description:** Allows users to stake XDC tokens inside their NFT to begin earning staking rewards.
* **Functionality:**
  * Users deposit XDC into their NFT to start accruing rewards.
  * Staking strengthens the NFT’s intrinsic value by tying rewards to ownership.

***

#### **2. Get Surplus**

* **Description:** Enables users to withdraw any excess XDC above 100,000 tokens from their NFT after reaching the top staking level.
* **Functionality:**
  * Users can retrieve surplus XDC that surpasses the maximum staking limit.
  * $pstXDC is **burned** proportionally to the amount of XDC withdrawn.
  * Ensures efficient fund management while keeping the NFT active.

***

#### **3. Withdraw XDC**

* **Description:** Users can withdraw their staked XDC tokens from the NFT, removing their staking position.
* **Functionality:**
  * Users **must burn** the corresponding $pstXDC when withdrawing their XDC.
  * Unlike the Get Surplus function, this completely removes the staked position but **does not burn the NFT**.
  * Withdrawn XDC is transferred back to the user’s wallet.

***

#### **4. Transfer**

* **Description:** Allows users to move their NFT between wallets.
* **Functionality:**
  * NFT ownership is updated on-chain when transferred to another user.
  * The new owner gains full control over the NFT and its staking benefits.

***

#### **5. Merge NFTs**

* **Description:** Combines two NFTs of the same rarity to create a higher-rarity NFT.
* **Functionality:**
  * Merging NFTs increases their rarity and enhances staking multipliers.
  * Both original NFTs are burned, and a new upgraded NFT is minted.

***

#### **6. Lock NFT**

* **Description:** Users can lock their NFT in a Prime Numbers XDC master node to receive additional staking rewards.
* **Functionality:**
  * Locked NFTs earn an additional 7% yearly staking rewards.
  * **Restriction:** Locked NFTs cannot use the Burn, Merge, or Withdraw features until unlocked.

***

#### **7. Claim XDC Rewards**

* **Description:** Users can claim their monthly staking rewards.
* **Functionality:**
  * Claiming rewards **mints $pstXDC** directly into the user’s wallet.
  * The equivalent amount of XDC rewards is **added to the NFT’s balance**.
  * Users must claim rewards manually each month.

***

#### **8. Sell**

* **Description:** Users can list or auction their XDC Staking NFT on the **PrimePort marketplace**.
* **Functionality:**
  * Sellers define the price and list their NFT for sale.
  * Buyers inherit the NFT’s staking rewards and features upon purchase.

***

#### **Security & Ownership Rules**

* Users must burn $pstXDC when withdrawing staked XDC.
* The NFT is **not burned** when withdrawing XDC, ensuring its continued usability.
* Locked NFTs cannot be merged, burned, or withdrawn until unlocked.
* Transactions are recorded on-chain, ensuring full transparency and ownership security.
