# ERC-721 Implementation from Scratch

## Simple Definition
Building an NFT from scratch helps you understand how unique IDs are assigned, how ownership is tracked per token, and how the approval mechanism works for transferring unique assets.

## The Best Analogy
Think of this like **issuing numbered tickets for a VIP event**. Each ticket has a unique number (Token ID). You keep a ledger (mapping) of who holds which ticket number. If someone wants to give their ticket to a friend, they must sign a permission slip (approve) or hand it over directly (transferFrom).

## Code Example: Minimal NFT

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MinimalNFT {
    string public name = "Minimal NFT";
    string public symbol = "MNFT";
    
    uint256 public totalSupply;
    uint256 private _nextTokenId;
    
    // Maps tokenId to its owner
    mapping(uint256 => address) public ownerOf;
    // Maps owner to their total NFT count
    mapping(address => uint256) public balanceOf;
    
    event Transfer(address indexed from, address indexed to, uint256 indexed tokenId);
    
    // Mint a new unique NFT to a specific address
    function mint(address to) public returns (uint256) {
        require(to != address(0), "Mint to zero address");
        
        uint256 tokenId = _nextTokenId++;
        totalSupply++;
        
        ownerOf[tokenId] = to;
        balanceOf[to]++;
        
        emit Transfer(address(0), to, tokenId);
        return tokenId;
    }
    
    // Transfer a specific NFT from one address to another
    function transferFrom(address from, address to, uint256 tokenId) public {
        require(ownerOf[tokenId] == from, "Not the owner");
        require(to != address(0), "Transfer to zero address");
        
        // Update ownership
        ownerOf[tokenId] = to;
        balanceOf[from]--;
        balanceOf[to]++;
        
        emit Transfer(from, to, tokenId);
    }
}
```

## Key Takeaways
- **Token ID Generation:** Using a simple counter (`_nextTokenId++`) is the easiest way to generate unique IDs. In production, you might use random numbers or specific patterns.
- **Ownership Mapping:** `mapping(uint256 => address)` is the core of an NFT. It tells you exactly who owns Token #5.
- **Zero Address Check:** Crucial for both minting and transferring to prevent losing NFTs forever.
- **Why use OpenZeppelin?** This minimal version lacks metadata (images/names), approvals, and safe transfers (`safeTransferFrom`). Always use OZ for real NFTs.