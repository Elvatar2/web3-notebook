# Damn Vulnerable DeFi: Free Rider

## Challenge Description
A new marketplace of Damn Valuable NFTs is being released. The marketplace offers a bounty of 45 ETH for anyone who can buy 6 specific NFTs and sell them back to the marketplace in the same transaction. The Uniswap pool has 15 WETH and 15 DVT. Your goal is to drain the marketplace's 45 ETH bounty.

**Difficulty:** ⭐⭐⭐⭐ (Hard)

**Goal:** Buy 6 NFTs and sell them back to the marketplace to claim the 45 ETH bounty, without using your own funds.

## Vulnerable Contracts

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract FreeRiderNFTMarketplace {
    IERC721 public token;
    uint256 public bounty;

    constructor(uint256 _bounty) payable {
        bounty = _bounty;
    }

    function offerMany(uint256[] calldata tokenIds, uint256[] calldata prices) external payable {
        require(tokenIds.length == prices.length, "Length mismatch");
        // ... logic to buy NFTs from users ...
    }

    function redeemMany(uint256[] calldata tokenIds) external {
        require(tokenIds.length == 6, "Must redeem 6 NFTs");
        // Check if msg.sender owns all 6 NFTs
        for (uint i = 0; i < 6; i++) {
            require(token.ownerOf(tokenIds[i]) == msg.sender, "Not owner");
        }
        // Transfer NFTs to marketplace and send bounty
        // ...
        payable(msg.sender).transfer(bounty);
    }
}
```

## Vulnerability Analysis

```text
THE BUG: The marketplace allows you to buy NFTs and immediately sell them back in the same transaction to claim the bounty.
However, you need the initial capital to buy the 6 NFTs from the Uniswap pool.

The solution is to use a Flash Loan!
1. Borrow WETH from a Flash Loan provider.
2. Use the WETH to buy the 6 NFTs from the Uniswap pool.
3. Approve the marketplace to take the NFTs.
4. Call redeemMany() to sell the NFTs back to the marketplace and receive the 45 ETH bounty.
5. Repay the Flash Loan with the bounty money.
6. Keep the remaining profit.
```

## Exploit Strategy

```text
Step 1: Deploy an attacker contract that implements the Flash Loan receiver interface.
Step 2: Request a Flash Loan for enough WETH to buy the 6 NFTs from Uniswap.
Step 3: In the executeFlashLoan callback:
        a. Swap WETH for DVT (or buy NFTs directly if the pool allows).
        b. Actually, in this specific challenge, you buy the NFTs from the Uniswap pool using WETH.
        c. Transfer the NFTs to the attacker contract.
        d. Approve the marketplace to transfer the NFTs.
        e. Call marketplace.redeemMany(tokenIds).
        f. The marketplace sends 45 ETH bounty to the attacker contract.
Step 4: Repay the Flash Loan.
Step 5: Send the remaining ETH to your EOA.
```

## Key Takeaways
- **Flash Loan Arbitrage:** Flash loans can be used to capitalize on bounties or price differences without initial capital.
- **Atomic Transactions:** The entire buy-sell-repay sequence happens in one transaction, making it risk-free (except for gas).
- **Bounty Exploits:** Always check if a bounty mechanism can be gamed by buying and selling the same asset in a single block.
- **Lesson Learned:** If a contract offers a reward for an action, ensure the action cannot be artificially manufactured using flash loans.
