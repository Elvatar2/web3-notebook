# ERC-20 Standard Overview

## Simple Definition
ERC-20 is the technical standard for fungible tokens on the Ethereum blockchain. It defines a common list of rules (functions and events) that all Ethereum tokens must implement to be considered "ERC-20 compliant".

## The Best Analogy
Think of ERC-20 like a **standardized USB-C port**. No matter if you are charging a phone, a laptop, or headphones, the plug fits and works the same way. Similarly, no matter if it's USDT, LINK, or UNI, wallets and exchanges know exactly how to interact with them because they all follow the ERC-20 "plug" standard.

## The ERC-20 Interface

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// This is the official interface that defines the ERC-20 standard
interface IERC20 {
    // 1. Total number of tokens in existence
    function totalSupply() external view returns (uint256);
    
    // 2. Get the token balance of a specific account
    function balanceOf(address account) external view returns (uint256);
    
    // 3. Transfer tokens from the caller's account to another
    function transfer(address to, uint256 amount) external returns (bool);
    
    // 4. Get the amount of tokens an owner allowed a spender to use
    function allowance(address owner, address spender) external view returns (uint256);
    
    // 5. Allow a spender to withdraw from your account, multiple times, up to the amount
    function approve(address spender, uint256 amount) external returns (bool);
    
    // 6. Transfer tokens from one account to another (requires prior approval)
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
    
    // 7. Event emitted when tokens are transferred
    event Transfer(address indexed from, address indexed to, uint256 value);
    
    // 8. Event emitted when approval is granted
    event Approval(address indexed owner, address indexed spender, uint256 value);
}
```

## Key Takeaways
- **Mandatory Functions:** `totalSupply`, `balanceOf`, `transfer`, `approve`, `transferFrom`, `allowance`.
- **Mandatory Events:** `Transfer` and `Approval` (crucial for wallets and block explorers to track activity).
- **Interoperability:** Because of this standard, MetaMask, Etherscan, and Uniswap can automatically detect and display any new ERC-20 token.
- **The "Allowance" Mechanism:** This is what allows smart contracts (like Uniswap) to spend your tokens on your behalf, but only up to the amount you explicitly `approve`.