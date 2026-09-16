# ERC-721 Standard Overview (NFTs)

## Simple Definition
ERC-721 is the standard for Non-Fungible Tokens (NFTs) on Ethereum. Unlike ERC-20 tokens where every token is identical (like dollars), every ERC-721 token is completely unique and has a distinct ID.

## The Best Analogy
If ERC-20 is like **paper money** (any $10 bill is worth the same as another $10 bill), ERC-721 is like **real estate or original paintings**. Every house has a unique address, and every painting has a unique signature. You cannot just swap one house for another and expect them to be equal in value.

## The ERC-721 Interface (Core Functions)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface IERC721 {
    // Returns the number of NFTs owned by an address
    function balanceOf(address owner) external view returns (uint256 balance);
    
    // Returns the owner of a specific NFT by its ID
    function ownerOf(uint256 tokenId) external view returns (address owner);
    
    // Transfers an NFT from one address to another
    function transferFrom(address from, address to, uint256 tokenId) external;
    
    // Approves another address to transfer a specific NFT
    function approve(address to, uint256 tokenId) external;
    
    // Events for tracking transfers and approvals
    event Transfer(address indexed from, address indexed to, uint256 indexed tokenId);
    event Approval(address indexed owner, address indexed approved, uint256 indexed tokenId);
}
```

## Key Takeaways
- **Token ID:** The most important concept. Every NFT is identified by a unique `uint256 tokenId`.
- **Non-Fungible:** Token #1 is completely different from Token #2. They cannot be divided or swapped 1:1.
- **Ownership Tracking:** Instead of just mapping `address => balance`, ERC-721 maps `tokenId => owner`.
- **Use Cases:** Digital art (Bored Apes), collectibles, real estate deeds, in-game items, and identity verification.