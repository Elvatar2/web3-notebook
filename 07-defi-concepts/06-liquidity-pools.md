# Liquidity Pools and LP Tokens

## Simple Definition
A liquidity pool is a smart contract that holds reserves of two or more tokens, enabling decentralized trading. Liquidity Providers (LPs) deposit tokens into these pools and receive "LP tokens" representing their share of the pool.

## The Best Analogy
Think of a liquidity pool like a **community potluck dinner**. Everyone brings food (tokens) to the common table (pool). In return, you get a "ticket" (LP token) representing your contribution. When people eat (trade), they pay a small fee that gets distributed to all ticket holders proportionally.

## Code Example: Liquidity Pool with LP Tokens

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract LiquidityPool {
    address public tokenA;
    address public tokenB;
    
    uint256 public reserveA;
    uint256 public reserveB;
    uint256 public totalLPTokens;
    
    mapping(address => uint256) public lpBalances;
    
    // Fee: 0.3% (3 out of 1000)
    uint256 public constant FEE_BPS = 30; // Basis points
    uint256 public constant FEE_DENOMINATOR = 10000;
    
    event LiquidityAdded(address provider, uint256 amountA, uint256 amountB, uint256 lpTokens);
    event LiquidityRemoved(address provider, uint256 amountA, uint256 amountB, uint256 lpTokens);
    event Swap(address trader, uint256 amountIn, uint256 amountOut, bool isAtoB);
    
    constructor(address _tokenA, address _tokenB) {
        tokenA = _tokenA;
        tokenB = _tokenB;
    }
    
    // Add liquidity to the pool
    function addLiquidity(uint256 amountA, uint256 amountB) public {
        require(amountA > 0 && amountB > 0, "Invalid amounts");
        
        uint256 lpTokensToMint;
        
        if (totalLPTokens == 0) {
            // First liquidity provider: use geometric mean
            lpTokensToMint = sqrt(amountA * amountB);
        } else {
            // Subsequent providers: proportional to existing reserves
            uint256 lpFromA = (amountA * totalLPTokens) / reserveA;
            uint256 lpFromB = (amountB * totalLPTokens) / reserveB;
            lpTokensToMint = min(lpFromA, lpFromB); // Use minimum to maintain ratio
        }
        
        require(lpTokensToMint > 0, "Insufficient LP tokens");
        
        // Update reserves
        reserveA += amountA;
        reserveB += amountB;
        totalLPTokens += lpTokensToMint;
        lpBalances[msg.sender] += lpTokensToMint;
        
        emit LiquidityAdded(msg.sender, amountA, amountB, lpTokensToMint);
    }
    
    // Remove liquidity from the pool
    function removeLiquidity(uint256 lpTokens) public {
        require(lpBalances[msg.sender] >= lpTokens, "Insufficient LP tokens");
        
        // Calculate proportional share of reserves
        uint256 amountA = (lpTokens * reserveA) / totalLPTokens;
        uint256 amountB = (lpTokens * reserveB) / totalLPTokens;
        
        // Update state
        lpBalances[msg.sender] -= lpTokens;
        totalLPTokens -= lpTokens;
        reserveA -= amountA;
        reserveB -= amountB;
        
        // Transfer tokens back to provider (simplified - would use ERC20 transfer)
        payable(msg.sender).transfer(amountA + amountB);
        
        emit LiquidityRemoved(msg.sender, amountA, amountB, lpTokens);
    }
    
    // Swap token A for token B
    function swapAforB(uint256 amountAIn) public payable {
        require(amountAIn > 0, "Invalid amount");
        
        // Apply fee
        uint256 amountAfterFee = amountAIn - (amountAIn * FEE_BPS / FEE_DENOMINATOR);
        
        // Calculate output using constant product formula
        uint256 amountBOut = (reserveB * amountAfterFee) / (reserveA + amountAfterFee);
        require(amountBOut < reserveB, "Insufficient liquidity");
        
        // Update reserves
        reserveA += amountAIn;
        reserveB -= amountBOut;
        
        emit Swap(msg.sender, amountAIn, amountBOut, true);
    }
    
    // Get LP's share of the pool
    function getLpShare(address provider) public view returns (uint256 shareA, uint256 shareB) {
        uint256 lpBalance = lpBalances[provider];
        shareA = (lpBalance * reserveA) / totalLPTokens;
        shareB = (lpBalance * reserveB) / totalLPTokens;
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
    
    receive() external payable {}
}
```

## How LP Tokens Work

```text
Example Scenario:
1. Alice deposits 10 ETH + 20,000 USDC → receives 100 LP tokens
2. Bob deposits 5 ETH + 10,000 USDC → receives 50 LP tokens
3. Total pool: 15 ETH + 30,000 USDC, 150 LP tokens

Trading happens:
- Traders pay 0.3% fees on each swap
- Fees accumulate in the pool (increase reserves)

After fees accumulate:
- Pool now has: 15.5 ETH + 30,500 USDC
- Alice's share: (100/150) * 15.5 = 10.33 ETH
- Alice's share: (100/150) * 30,500 = 20,333 USDC
- Alice earned: 0.33 ETH + 333 USDC in fees!

When Alice removes liquidity:
- She burns her 100 LP tokens
- Receives her proportional share: 10.33 ETH + 20,333 USDC
```

## Key Takeaways
- **LP Tokens = Receipt:** They prove your ownership share of the pool.
- **Proportional Share:** Your LP tokens represent a percentage of total pool.
- **Fee Accumulation:** As trades happen, the pool grows, increasing your share's value.
- **Burning LP Tokens:** When you withdraw, LP tokens are burned (destroyed).
- **Impermanent Loss:** If token prices change significantly, you may have been better off just holding (see file 07).
- **Composability:** LP tokens can be used as collateral in other DeFi protocols (e.g., borrowing against them).