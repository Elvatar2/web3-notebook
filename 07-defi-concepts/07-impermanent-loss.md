# Impermanent Loss (IL)

## Simple Definition
Impermanent Loss (IL) is the difference in value between holding tokens in a liquidity pool versus simply holding them in your wallet. It occurs when the price ratio of the two tokens changes after you deposit them. It's called "impermanent" because if prices return to the original ratio, the loss disappears.

## The Best Analogy
Think of IL like **betting on a horse race where you must own both horses**. If Horse A wins (price goes up), you're happy, but you were forced to sell some of Horse A to buy more of Horse B (which lost). You end up with less of the winning asset than if you had just held both separately. The "impermanent" part is like saying "the race isn't over yet" - if the horses tie again, your loss vanishes.

## Mathematical Example

```text
Initial Deposit:
- 1 ETH @ $2000 + 2000 USDC = $4000 total
- Pool ratio: 1 ETH : 2000 USDC

Scenario: ETH price doubles to $4000

If you just held (HODL):
- 1 ETH @ $4000 + 2000 USDC = $6000
- Profit: $2000 (50% gain)

If you provided liquidity:
- Pool rebalances to maintain x*y=k
- New pool: ~0.707 ETH + ~2828 USDC = $5656
- Profit: $1656 (41.4% gain)

Impermanent Loss:
- IL = $6000 - $5656 = $344
- IL % = 5.7% of portfolio value
```

## Code Example: Calculating Impermanent Loss

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract ImpermanentLossCalculator {
    
    // Calculate IL given price change ratio
    // priceRatio = newPrice / oldPrice (e.g., 2.0 for 2x increase)
    function calculateIL(uint256 priceRatio) public pure returns (int256 ilPercentage) {
        require(priceRatio > 0, "Invalid ratio");
        
        // Formula: IL = 2*sqrt(priceRatio)/(1+priceRatio) - 1
        // Simplified using fixed-point math (18 decimals)
        
        uint256 sqrtRatio = sqrt(priceRatio * 1e18);
        uint256 numerator = 2 * sqrtRatio;
        uint256 denominator = 1e18 + priceRatio;
        
        uint256 result = (numerator * 1e18) / denominator;
        
        // Convert to percentage with sign
        if (result < 1e18) {
            ilPercentage = -int256(1e18 - result); // Negative = loss
        } else {
            ilPercentage = int256(result - 1e18); // Positive = gain (rare)
        }
    }
    
    // Compare HODL vs LP value
    function compareHodlVsLP(
        uint256 initialEth,
        uint256 initialUsdc,
        uint256 priceChangeMultiplier
    ) public pure returns (uint256 hodlValue, uint256 lpValue, int256 loss) {
        // HODL value
        hodlValue = (initialEth * priceChangeMultiplier) + initialUsdc;
        
        // LP value (using constant product formula)
        uint256 k = initialEth * initialUsdc;
        uint256 newEth = sqrt((k * 1e18) / priceChangeMultiplier);
        uint256 newUsdc = k / newEth;
        lpValue = (newEth * priceChangeMultiplier) + newUsdc;
        
        loss = int256(lpValue) - int256(hodlValue);
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
}
```

## IL Percentage Table

```text
Price Change | Impermanent Loss
-------------|------------------
1.25x        | 0.6%
1.50x        | 2.0%
1.75x        | 3.8%
2.00x        | 5.7%
3.00x        | 13.4%
4.00x        | 20.0%
5.00x        | 25.5%
```

## How to Mitigate IL

```text
1. Stablecoin Pairs:
   - USDC/USDT, DAI/USDC have minimal IL (prices stay close)
   - IL typically < 0.1%
   
2. High Trading Fees:
   - If fees earned > IL, you're profitable
   - Volatile pairs need higher fee tiers
   
3. Impermanent Loss Protection:
   - Some protocols (like Bancor) offer IL protection
   - After X days, protocol covers your IL
   
4. Single-Sided Staking:
   - Provide only one asset (no IL risk)
   - Lower returns but safer
   
5. Concentrated Liquidity (V3):
   - Can reduce IL by narrowing range
   - But increases risk of going out of range
```

## Key Takeaways
- **IL is Unavoidable:** Any time prices diverge, IL occurs in AMM pools.
- **Fees vs. IL:** You need trading fees to exceed IL to be profitable.
- **Worse with Volatility:** More price change = more IL.
- **Permanent if Withdrawn:** IL becomes permanent when you withdraw at a different price ratio.
- **Stable Pairs are Safe:** Stablecoin pairs have minimal IL.
- **Not Really "Impermanent":** The name is misleading - it becomes permanent upon withdrawal.