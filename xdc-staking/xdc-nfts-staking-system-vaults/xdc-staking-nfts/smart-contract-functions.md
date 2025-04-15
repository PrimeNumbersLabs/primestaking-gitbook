# Smart Contract Functions

**Core functions within the XDC Staking NFTs smart contract, defining their purpose and usage.**

***

1. **Stake**
   * **Description**: Allows users to stake **$psXDC** inside their NFT to begin earning NFT-based rewards.
   * **Functionality**:\
     Users deposit **$psXDC** into their NFT; this increases the NFT’s intrinsic value and starts accruing NFT-level rewards.
2. **Withdraw $psXDC**
   * **Description**: Users can withdraw their staked **$psXDC** tokens from the NFT, partially or fully removing their staking position.
   * **Functionality**:\
     When withdrawing, the corresponding **$psXDC** is sent to the user, symbolizing the redemption of the underlying stake.\
     This does not burn the NFT, allowing it to remain in use or be restaked later.\
     Withdrawn tokens are returned to the user’s wallet in **$psXDC** form.
3. **Transfer**
   * **Description**: Allows users to move their NFT between different wallets.
   * **Functionality**:\
     On transfer, NFT ownership is updated on-chain.\
     The new owner inherits the NFT’s staked **$psXDC** and associated reward multipliers, as well as any locked status or available surplus.
4. **Merge NFTs**
   * **Description**: Combines two NFTs of the same rarity to create a higher-rarity NFT.
   * **Functionality**:\
     Both original NFTs are burned, and a new, upgraded NFT is minted with improved base multipliers for staking.\
     The newly created NFT can stake **$psXDC** at an enhanced rate, potentially increasing overall rewards.
5. **Lock NFT**
   * **Description**: Users can lock their NFT for an additional yield (for instance, 7.25% + variable APY) over a specified period.
   * **Functionality**:\
     Locked NFTs earn extra staking rewards, typically paid monthly.\
     **Restriction**: While locked, the NFT cannot use Merge, Withdraw, or other functions that would alter its staked position.
6. **Claim $XDC Rewards**
   * **Description**: Users can claim their monthly rewards generated through the NFT’s multipliers and staked **$psXDC**. Rewards are paid in **$XDC**.
7. **Sell**
   * **Description**: Users can list or auction their XDC Staking NFT on the PrimePort marketplace (or any compatible NFT marketplace).
   * **Functionality**:\
     Sellers set their price and transfer NFT ownership to the buyer upon sale.\
     The buyer then acquires the NFT’s staked **$psXDC**, any pending rewards, and the right to future yields.
