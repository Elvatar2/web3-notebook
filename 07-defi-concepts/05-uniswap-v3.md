# Uniswap V3: Concentrated Liquidity

## Simple Definition
Uniswap V3 introduced "concentrated liquidity," allowing liquidity providers to specify custom price ranges for their capital. Instead of spreading liquidity across all prices (0 to infinity like V2), LPs can concentrate their funds where most trading happens, earning more fees with less capital.

## The Best Analogy
Think of Uniswap V2 like **spreading butter evenly across the entire bread** - most of it goes to waste on the edges. Uniswap V3 is like **putting butter only where you'll actually bite** - concentrated exactly where it's needed, so you use less butter but get the same coverage where it matters.

## Code Example: Concentrated Liquidity Position

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// Simplified representation of a V3-style concentrated liquidity position
contract ConcentratedLiquidity {
    struct Position {
        address owner;
        int24 tickLower;  // Lower price boundary
        int24 tickUpper;  // Upper price boundary
        uint128 liquidity; // Amount of liquidity
        uint256 tokensOwed0; // Fees earned in token0
        uint256 tokensOwed1; // Fees earned in token1
    }
    
    mapping(uint256 => Position) public positions;
    uint256 public nextPositionId;
    
    uint256 public reserve0;
    uint256 public reserve1;
    int24 public currentTick;
    
    // Add liquidity within a specific price range
    function mintPosition(
        int24 _tickLower,
        int24 _tickUpper,
        uint128 _liquidity
    ) public returns (uint256 positionId) {
        require(_tickLower < _tickUpper, "Invalid range");
        require(_liquidity > 0, "Zero liquidity");
        
        positionId = nextPositionId++;
        
        positions[positionId] = Position({
            owner: msg.sender,
            tickLower: _tickLower,
            tickUpper: _tickUpper,
            liquidity: _liquidity,
            tokensOwed0: 0,
            tokensOwed1: 0
        });
        
        // If current price is within range, add to active reserves
        if (currentTick >= _tickLower && currentTick < _tickUpper) {
            // Calculate token amounts needed (simplified)
            uint256 amount0 = (_liquidity * 1e18) / (1e12); // Simplified math
            uint256 amount1 = (_liquidity * 1e12) / 1e18;
            
            reserve0 += amount0;
            reserve1 += amount1;
        }
    }
    
    // Check if a position is active (current price in range)
    function isPositionActive(uint256 positionId) public view returns (bool) {
        Position storage pos = positions[positionId];
        return currentTick >= pos.tickLower && currentTick < pos.tickUpper;
    }
    
    // Collect fees earned by a position
    function collectFees(uint256 positionId) public {
        Position storage pos = positions[positionId];
        require(msg.sender == pos.owner, "Not owner");
        
        uint256 owed0 = pos.tokensOwed0;
        uint256 owed1 = pos.tokensOwed1;
        
        pos.tokensOwed0 = 0;
        pos.tokensOwed1 = 0;
        
        // Transfer fees to owner (simplified)
        payable(msg.sender).transfer(owed0 + owed1);
    }
}
```

## V2 vs V3 Comparison

```text
Uniswap V2:
- Liquidity spread from price 0 to 
- Capital inefficient (most liquidity unused)
- Simple to understand
- LP earns 0.3% on all trades in pool

Uniswap V3:
- Liquidity concentrated in custom ranges
- 4000x more capital efficient (theoretically)
- Complex to manage (need to rebalance)
- LP earns fees ONLY when price is in their range
- Multiple fee tiers: 0.05%, 0.3%, 1%
```

## Key Takeaways
- **Capital Efficiency:** V3 allows earning same fees with 100x less capital if positioned correctly.
- **Active Management:** LPs must monitor and rebalance positions as price moves.
- **Impermanent Loss Amplified:** IL can be worse in V3 if price exits your range.
- **Fee Tiers:** Different pairs use different fee tiers based on volatility.
- **Ticks:** Price ranges are defined by "ticks" (discrete price points).
- **Complexity:** V3 is much more complex than V2, requiring advanced knowledge.