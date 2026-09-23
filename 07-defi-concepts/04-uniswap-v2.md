# Uniswap V2 Architecture

## Simple Definition
Uniswap V2 is the most popular AMM-based decentralized exchange on Ethereum. It uses a system of "Pair" contracts (one for each token pair) and a "Router" contract (for user-friendly trading).

## The Best Analogy
Think of Uniswap V2 like a **network of specialized exchange booths**. Each booth (Pair contract) only trades one specific pair (e.g., ETH/USDC). The main desk (Router contract) helps you find the right booth and execute your trade, even if you need to go through multiple booths (multi-hop swaps).

## Uniswap V2 Architecture

```text
User
  |
  v
[Router Contract] <-- User interacts with this
  |
  +---> [ETH/USDC Pair] <-- Liquidity pool for ETH/USDC
  |         |
  |         +---> LPs earn 0.3% fees on every trade
  |
  +---> [ETH/DAI Pair] <-- Another liquidity pool
  |
  +---> [USDC/DAI Pair] <-- Can do multi-hop: ETH -> USDC -> DAI
```

## Code Example: Uniswap V2 Pair Contract (Simplified)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// Simplified version of UniswapV2Pair
contract UniswapV2Pair {
    address public token0; // First token (e.g., USDC)
    address public token1; // Second token (e.g., ETH)
    
    uint256 public reserve0;
    uint256 public reserve1;
    uint256 public totalSupply; // LP tokens
    
    mapping(address => uint256) public balanceOf; // LP token balances
    
    uint256 public constant FEE_DENOMINATOR = 1000;
    uint256 public constant FEE_NUMERATOR = 3; // 0.3% fee
    
    constructor(address _token0, address _token1) {
        token0 = _token0;
        token1 = _token1;
    }
    
    // Add liquidity and receive LP tokens
    function addLiquidity(uint256 amount0, uint256 amount1) public {
        require(amount0 > 0 && amount1 > 0, "Invalid amounts");
        
        uint256 liquidity;
        if (totalSupply == 0) {
            // First liquidity provider
            liquidity = sqrt(amount0 * amount1);
        } else {
            // Subsequent liquidity providers
            liquidity = min(
                (amount0 * totalSupply) / reserve0,
                (amount1 * totalSupply) / reserve1
            );
        }
        
        require(liquidity > 0, "Insufficient liquidity");
        
        reserve0 += amount0;
        reserve1 += amount1;
        totalSupply += liquidity;
        balanceOf[msg.sender] += liquidity;
    }
    
    // Swap token0 for token1
    function swap(uint256 amount0Out, uint256 amount1Out) public {
        require(amount0Out > 0 || amount1Out > 0, "Invalid output");
        require(amount0Out < reserve0 && amount1Out < reserve1, "Insufficient liquidity");
        
        // Calculate required input with 0.3% fee
        uint256 amount0In = amount0Out > 0 
            ? (reserve0 * amount0Out * FEE_DENOMINATOR) / ((reserve0 - amount0Out) * (FEE_DENOMINATOR - FEE_NUMERATOR)) + 1
            : 0;
            
        uint256 amount1In = amount1Out > 0
            ? (reserve1 * amount1Out * FEE_DENOMINATOR) / ((reserve1 - amount1Out) * (FEE_DENOMINATOR - FEE_NUMERATOR)) + 1
            : 0;
        
        // Update reserves
        reserve0 += amount0In;
        reserve1 += amount1In;
        reserve0 -= amount0Out;
        reserve1 -= amount1Out;
        
        // Verify k increased (accounting for fees)
        require(reserve0 * reserve1 >= totalSupply * totalSupply, "K value decreased");
        
        // Transfer tokens to user (simplified)
    }
    
    // Remove liquidity and get tokens back
    function removeLiquidity(uint256 liquidity) public {
        require(balanceOf[msg.sender] >= liquidity, "Insufficient LP tokens");
        
        uint256 amount0 = (liquidity * reserve0) / totalSupply;
        uint256 amount1 = (liquidity * reserve1) / totalSupply;
        
        balanceOf[msg.sender] -= liquidity;
        totalSupply -= liquidity;
        reserve0 -= amount0;
        reserve1 -= amount1;
        
        // Transfer tokens to user (simplified)
    }
    
    function sqrt(uint256 y) internal pure returns (uint256 z) {
        if (y > 3) {
            z = y;
            uint256 x = y / 2 + 1;
            while (x < z) {
                z = x;
                x = (y / x + x) / 2;
            }
        } else if (y != 0) {
            z = 1;
        }
    }
    
    function min(uint256 a, uint256 b) internal pure returns (uint256) {
        return a < b ? a : b;
    }
}
```

## How a Trade Works (Step-by-Step)

```text
User wants to swap 1 ETH for USDC:

1. User calls Router.swapExactETHForTokens()
2. Router sends 1 ETH to ETH/USDC Pair contract
3. Pair contract calculates USDC output using x*y=k formula
4. Pair contract applies 0.3% fee (0.003 ETH stays in pool)
5. Pair contract sends USDC to user
6. LPs earn the 0.3% fee (distributed proportionally)

Example with numbers:
- Pool: 100 ETH, 200,000 USDC (k = 20,000,000)
- User sends: 1 ETH
- Fee: 0.003 ETH (stays in pool)
- Effective input: 0.997 ETH
- New reserve ETH: 100.997
- New reserve USDC: 20,000,000 / 100.997 = 198,025.68
- User receives: 200,000 - 198,025.68 = 1,974.32 USDC
- Price per ETH: 1,974.32 USDC (slightly less than 2,000 due to slippage)
```

## Key Takeaways
- **Pair Contracts:** Each token pair has its own contract holding liquidity.
- **Router Contract:** User-friendly interface for swaps, adds, and removes.
- **LP Tokens:** Represent your share of the liquidity pool.
- **0.3% Fee:** Charged on every trade, distributed to LPs.
- **Constant Product:** x*y=k ensures liquidity but causes slippage.
- **Multi-hop Swaps:** Router can route trades through multiple pairs for better prices.
- **Flash Swaps:** Uniswap V2 allows borrowing tokens as long as you return them in the same transaction.