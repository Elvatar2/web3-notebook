# ERC-1155 Multi-Token Standard

## Simple Definition
ERC-1155 is a multi-token standard that allows a single contract to manage multiple types of tokens (both fungible and non-fungible) efficiently. Unlike ERC-20 (one contract per token) or ERC-721 (one contract per NFT collection), ERC-1155 can handle thousands of different tokens in one contract.

## The Best Analogy
Think of ERC-1155 like a **department store** versus individual shops:
- **ERC-20:** Each product has its own separate store (one contract per token).
- **ERC-721:** Each unique item has its own boutique (one contract per NFT collection).
- **ERC-1155:** One big department store with multiple floors selling everything - groceries (fungible tokens), electronics (semi-fungible), and art (NFTs) - all under one roof (one contract).

## Key Differences from ERC-20 and ERC-721

| Feature | ERC-20 | ERC-721 | ERC-1155 |
|---------|--------|---------|----------|
| **Token Type** | Fungible only | Non-fungible only | Both (Fungible + NFT) |
| **Contract Efficiency** | One contract per token | One contract per collection | One contract for ALL tokens |
| **Batch Transfers** | No | No | Yes (save gas!) |
| **Gas Cost** | Medium | High | Low (batched) |
| **Use Case** | Currencies, utility tokens | Art, collectibles | Gaming items, tickets, bundles |

## The ERC-1155 Interface (Core Functions)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface IERC1155 {
    // Get balance of a specific token ID for an address
    function balanceOf(address account, uint256 id) external view returns (uint256);
    
    // Transfer a single token type
    function safeTransferFrom(address from, address to, uint256 id, uint256 amount, bytes calldata data) external;
    
    // Transfer multiple token types in one transaction (BATCH!)
    function safeBatchTransferFrom(address from, address to, uint256[] calldata ids, uint256[] calldata amounts, bytes calldata data) external;
    
    // Events
    event TransferSingle(address indexed operator, address indexed from, address indexed to, uint256 id, uint256 value);
    event TransferBatch(address indexed operator, address indexed from, address indexed to, uint256[] ids, uint256[] values);
}
```

## Key Takeaways
- **Token IDs:** Each token type has a unique ID. ID #1 could be "Gold Coins" (fungible, supply = 1000000), while ID #2 could be "Legendary Sword #5" (NFT, supply = 1).
- **Batch Operations:** You can transfer 10 different items in ONE transaction, saving massive amounts of gas.
- **Gaming Use Case:** Perfect for games where players have multiple items (potions, swords, armor) - all managed in one contract.
- **Supply Flexibility:** Some IDs can have unlimited supply (fungible), others can have supply = 1 (NFTs).
- **Efficiency:** Much cheaper to deploy and interact with than multiple ERC-20 or ERC-721 contracts.