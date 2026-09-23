# Automated Market Makers (AMM)

## Simple Definition
An Automated Market Maker (AMM) is a type of decentralized exchange that uses mathematical formulas (instead of order books) to price assets. Liquidity providers deposit tokens into pools, and traders trade against these pools.

## The Best Analogy
Think of an AMM like a **currency exchange booth at an airport**. The booth has a fixed amount of USD and EUR. If you want to buy EUR with USD, the booth automatically calculates the exchange rate based on how much of each currency is left. As more people buy EUR, the price of EUR goes up (because there's less EUR in the booth).

## The Constant Product Formula: x * y = k

```text
Where:
- x = Amount of Token A in the pool
- y = Amount of Token B in the pool
- k = Constant product (must remain constant after trades)

Example:
- Pool has 100 ETH (x) and 200,000 USDC (y)
- k = 100 * 200,000 = 20,000,000

If someone buys 1 ETH:
- New x = 100 - 1 = 99 ETH
- New y = k / new x = 20,000,000 / 99 = 202,020.20 USDC
- User pays: 202,020.20 - 200,000 = 2,020.20 USDC for 1 ETH
- Price per ETH: 2,020.20 USDC (slightly higher than initial 2,000 USDC)
```

## Code Example: Simple AMM Implementation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleAMM {
    uint256 public reserveA; // Token A (e.g., ETH)
    uint256 public reserveB; // Token B (e.g., USDC)
    uint256 public constant K; // Constant product
    
    constructor(uint256 _reserveA, uint256 _reserveB) {
        reserveA = _reserveA;
        reserveB = _reserveB;
        K = _reserveA * _reserveB; // k = x * y
    }
    
    // Calculate how much Token B you get for Token A
    function getSwapOutput(uint256 amountIn) public view returns (uint256) {
        require(amountIn > 0, "Amount must be > 0");
        
        // Formula: output = (reserveB * amountIn) / (reserveA + amountIn)
        uint256 numerator = reserveB * amountIn;
        uint256 denominator = reserveA + amountIn;
        return numerator / denominator;
    }
    
    // Swap Token A for Token B
    function swapAforB(uint256 amountA) public payable {
        require(msg.value == amountA, "Incorrect ETH amount");
        
        uint256 amountB = getSwapOutput(amountA);
        require(amountB > 0, "Output too small");
        
        // Update reserves
        reserveA += amountA;
        reserveB -= amountB;
        
        // Verify k is maintained (with small rounding tolerance)
        require(reserveA * reserveB >= K, "K value decreased");
        
        // Send Token B to user (simplified - in real AMM, use ERC20 transfer)
        payable(msg.sender).transfer(amountB);
    }
    
    // Add liquidity (become a liquidity provider)
    function addLiquidity(uint256 amountA, uint256 amountB) public payable {
        require(msg.value == amountA, "Incorrect ETH amount");
        
        reserveA += amountA;
        reserveB += amountB;
        
        // In real AMM, mint LP tokens here
    }
    
    // Get current price of Token A in terms of Token B
    function getPrice() public view returns (uint256) {
        return reserveB / reserveA;
    }
    
    receive() external payable {}
}
```

## Understanding Slippage

```text
Slippage = Difference between expected price and actual price

Example:
- You want to buy 10 ETH
- Expected price: 2,000 USDC per ETH
- Expected cost: 20,000 USDC

But due to large trade size:
- Actual price: 2,100 USDC per ETH (price moved)
- Actual cost: 21,000 USDC
- Slippage: 1,000 USDC (5%)

Solutions:
1. Set maximum slippage tolerance (e.g., 0.5%)
2. Split large trades into smaller ones
3. Use DEX aggregators (1inch, Paraswap) for better prices
```

## Key Takeaways
- **No Order Book:** AMMs use mathematical formulas instead of matching buyers/sellers.
- **Liquidity Providers (LPs):** Users who deposit tokens earn trading fees (typically 0.3% per trade).
- **Constant Product:** The formula x*y=k ensures there's always liquidity, but causes slippage.
- **Price Impact:** Large trades move the price more (higher slippage).
- **Impermanent Loss:** LPs can lose money if token prices change significantly (covered in file 07).
- **Innovation:** Uniswap V3 introduced "concentrated liquidity" for better capital efficiency.