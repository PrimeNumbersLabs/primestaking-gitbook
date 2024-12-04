---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Staking Design

<mark style="color:red;">**Note:**</mark> <mark style="color:red;"></mark><mark style="color:red;">Our NFTs are being migrated to a new network.</mark>&#x20;

### <mark style="color:blue;">Smart-contracts:</mark>

* PrimeDiamond: Implements a diamond contract model (following EIPs 2535 standards).
* AdminFacet: Manages administrative duties such as modifying withdrawal charges, authorizing specific addresses, and initiating new staking interfaces.
* StakingFacet: Facilitates the migration, consolidation, staking, and retrieval of particular NFTs.
* ERC721Facet: Allows for the creation, destruction, exchange, and authorization of NFTs.
* StakingGetter: Retrieves all data functions stored within the contract.

***

### <mark style="color:blue;">Benefits of Employing This Model:</mark>

* Upgradeability: The diamond protocol enhances the efficiency of contract upgrades.
* Flexibility: This model enables associating multiple assets with a single address.
* Security: Contracts can be suspended to mitigate risks in the event of identified vulnerabilities.
* User Convenience: Users can manage their NFTs seamlessly without transferring contracts anew, as all information is consolidated within the diamond contract.
* Scalability: The contract is designed to incorporate additional features and functionalities without impacting the core protocol.\


EIP 2535 is the foundational protocol standard in the updated Prime Numbers Staking. It utilizes the diamond standard alongside a bespoke staking interface standard for ERC-721 NFTs to manage asset interactions within the protocol.&#x20;

The ERC-20 standard is utilized for asset management (excluding XDC). The system incorporates a re-entrance guard to thwart re-entrance assaults and employs the Create2 Yul opcode to deploy staking interfaces. The ABDKMathQuad library calculates NFT rewards.&#x20;

The interface is integral to a broader staking and ERC721 token system, with specific functionalities dependent on the Staking Facets and Staking Getter contract implementations.

\
