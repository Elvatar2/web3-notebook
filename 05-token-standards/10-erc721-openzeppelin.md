# ERC-721 with OpenZeppelin

## Simple Definition
Using OpenZeppelin's ERC-721 implementation provides a secure, gas-optimized, and feature-complete NFT contract. It includes safe transfers, metadata support, and enumeration (counting total NFTs) out of the box.

## The Best Analogy
Think of OpenZeppelin's ERC-721 like buying a **pre-fabricated, code-certified house**. The foundation, walls, and roof are already built and inspected by experts. You just need to add your personal touches (custom functions) and decorate it (mint your NFTs).

## Code Example: Production-Ready NFT

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Burnable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract ProductionNFT is ERC721, ERC721URIStorage, ERC721Burnable, Ownable {
    uint256 private _nextTokenId;
    uint256 public constant MAX_SUPPLY = 10000;
    uint256 public mintPrice = 0.05 ether;
    
    constructor() ERC721("Production NFT", "PNFT") Ownable(msg.sender) {}
    
    // Public mint function with price and supply limit
    function safeMint(uint256 quantity) public payable {
        require(_nextTokenId + quantity <= MAX_SUPPLY, "Would exceed max supply");
        require(msg.value >= mintPrice * quantity, "Insufficient payment");
        
        for (uint256 i = 0; i < quantity; i++) {
            uint256 tokenId = _nextTokenId++;
            _safeMint(msg.sender, tokenId);
            _setTokenURI(tokenId, generateTokenURI(tokenId));
        }
    }
    
    // Generate metadata URI (could point to IPFS or a server)
    function generateTokenURI(uint256 tokenId) internal pure returns (string memory) {
        return string.concat(
            "ipfs://QmYourBaseURI/",
            vm.toString(tokenId),
            ".json"
        );
    }
    
    // Withdraw ETH collected from mints
    function withdraw() public onlyOwner {
        (bool success, ) = payable(owner()).call{value: address(this).balance}("");
        require(success, "Withdraw failed");
    }
    
    function tokenURI(uint256 tokenId) public view override(ERC721, ERC721URIStorage) returns (string memory) {
        return super.tokenURI(tokenId);
    }
}
```

## Key Takeaways
- **ERC721URIStorage:** Extension that enables storing and updating token URIs.
- **ERC721Burnable:** Allows token holders to burn their NFTs.
- **_safeMint:** Ensures the recipient is a valid address (prevents sending to contracts that can't handle NFTs).
- **Supply Limits:** Always implement a `MAX_SUPPLY` to prevent infinite minting.
- **Payable Mint:** Real NFT projects charge ETH for minting; always validate `msg.value`.
- **Base URI:** You can set a base URI (`_setBaseURI()`) and append token IDs to it for efficient storage.