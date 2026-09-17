# ERC-721 Metadata and IPFS

## Simple Definition
ERC-721 Metadata is the information that describes an NFT (name, description, image, attributes). Since storing large files like images directly on the blockchain is extremely expensive, we store the metadata on IPFS (InterPlanetary File System) and only store the IPFS hash (URI) on-chain.

## The Best Analogy
Think of an NFT like a **museum plaque**. The plaque on the wall (on-chain data) has a reference number (Token URI) that points to a detailed catalog in the library (IPFS). The catalog contains the full description, high-resolution photo, and artist biography. The plaque is permanent and unchangeable, but the catalog can be accessed by anyone who has the reference number.

## The ERC-721 Metadata Interface

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface IERC721Metadata {
    function name() external view returns (string memory);
    function symbol() external view returns (string memory);
    function tokenURI(uint256 tokenId) external view returns (string memory);
}
```

## Metadata JSON Structure

```json
{
  "name": "Bored Ape #1234",
  "description": "A unique NFT in the Bored Ape collection",
  "image": "ipfs://QmYxT4LnK8sqLupjbS6eRvu1si7Ly2wFQAqFebxhWntcf6",
  "attributes": [
    {
      "trait_type": "Background",
      "value": "Blue"
    },
    {
      "trait_type": "Fur",
      "value": "Golden Brown"
    },
    {
      "trait_type": "Eyes",
      "value": "Bored"
    }
  ]
}
```

## Code Example: Setting Metadata

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";

contract MyNFT is ERC721, ERC721URIStorage {
    uint256 private _nextTokenId;
    
    constructor() ERC721("My NFT", "MNFT") {}
    
    function mint(address to) public returns (uint256) {
        uint256 tokenId = _nextTokenId++;
        _safeMint(to, tokenId);
        
        // Set the metadata URI for this specific token
        string memory uri = "ipfs://QmYourMetadataHashHere";
        _setTokenURI(tokenId, uri);
        
        return tokenId;
    }
    
    function tokenURI(uint256 tokenId) public view override(ERC721, ERC721URIStorage) returns (string memory) {
        return super.tokenURI(tokenId);
    }
}
```

## Key Takeaways
- **Off-Chain Storage:** Images and large metadata are stored on IPFS, not on the blockchain.
- **Token URI:** The smart contract only stores a link (URI) to the metadata JSON file.
- **Immutability:** Once minted, the URI should ideally never change to maintain trust.
- **IPFS Gateway:** Users access IPFS content via gateways like `https://gateway.pinata.cloud/ipfs/{hash}`.
- **Attributes:** The `attributes` array in metadata is what marketplaces like OpenSea use to display traits and rarity.