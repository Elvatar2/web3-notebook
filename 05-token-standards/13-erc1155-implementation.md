# ERC-1155 Implementation

## Simple Definition
Implementing an ERC-1155 contract allows you to create a unified system for both fungible (currencies, resources) and non-fungible (unique items, NFTs) tokens within a single smart contract.

## The Best Analogy
Think of an ERC-1155 contract like a **video game inventory system**. Instead of having one database for "Gold Coins" and a completely separate database for "Unique Swords", the game uses one master inventory list. Item ID #1 is Gold (you can have 999 of them), and Item ID #2 is a Legendary Sword (you can only have 1).

## Code Example: Game Items Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract GameItems is ERC1155, Ownable {
    uint256 public constant GOLD = 1;
    uint256 public constant SILVER = 2;
    uint256 public constant LEGENDARY_SWORD = 3;
    
    // Base URI for metadata (e.g., "https://game.com/api/item/{id}.json")
    string private _baseURI;

    constructor(string memory baseURI) ERC1155(baseURI) Ownable(msg.sender) {
        _baseURI = baseURI;
        
        // Mint initial supply of fungible items to the owner
        _mint(msg.sender, GOLD, 1000000 * 10**18, "");
        _mint(msg.sender, SILVER, 500000 * 10**18, "");
        
        // Mint a single non-fungible item (NFT) to the owner
        _mint(msg.sender, LEGENDARY_SWORD, 1, "");
    }

    // Allow owner to mint more items
    function mint(address account, uint256 id, uint256 amount) public onlyOwner {
        _mint(account, id, amount, "");
    }

    // Required override to support the base URI with {id} placeholder
    function uri(uint256) public view override returns (string memory) {
        return _baseURI;
    }
}
```

## Key Takeaways
- **Single Contract:** Manages multiple token types efficiently.
- **Fungible vs. Non-Fungible:** Determined by the `amount` parameter. Amount > 1 is fungible; Amount == 1 is an NFT.
- **URI Placeholder:** The `{id}` in the base URI is automatically replaced by the token ID by wallets and marketplaces.
- **Batch Operations:** Inherits `safeBatchTransferFrom` from OpenZeppelin, allowing users to send multiple item types in one transaction.