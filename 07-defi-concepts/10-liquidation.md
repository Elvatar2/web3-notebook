# Liquidation in DeFi

## Simple Definition
Liquidation is the process where a DeFi protocol automatically sells a borrower's collateral to repay their debt when the Health Factor drops below 1.0. This protects the protocol from becoming insolvent (having more debt than assets).

## The Best Analogy
Think of liquidation like a **margin call in traditional stock trading**. If your stock drops too much and your account equity falls below the broker's requirement, the broker automatically sells your stocks to cover the loan, without asking for your permission. In DeFi, "liquidators" (bots or users) do this selling, and they get a "liquidation bonus" as a reward.

## How Liquidation Works (Step-by-Step)

```text
1. User borrows $10,000 USDC against $15,000 worth of ETH.
2. ETH price crashes. The collateral is now worth only $11,000.
3. Health Factor drops below 1.0 (e.g., 0.9).
4. A Liquidator bot detects this unhealthy position.
5. Liquidator repays a portion of the user's debt (e.g., $5,000 USDC) to the protocol.
6. In return, the protocol gives the Liquidator $5,000 worth of ETH PLUS a 5-10% bonus.
7. User's debt is reduced, and their remaining collateral is returned to them.
```

## Code Example: Liquidation Function

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract LiquidationEngine {
    mapping(address => uint256) public collateralEth; // in Wei
    mapping(address => uint256) public debtUsdc;      // in USDC units
    
    uint256 public ethPrice = 2000; // Simplified oracle price
    uint256 public constant LIQUIDATION_THRESHOLD = 75; // 75%
    uint256 public constant LIQUIDATION_BONUS = 5; // 5% bonus for liquidator
    uint256 public constant MAX_LIQUIDATABLE_PERCENT = 50; // Max 50% of debt can be liquidated at once

    event Liquidated(address borrower, address liquidator, uint256 debtRepaid, uint256 collateralSeized);

    // Check if position is unhealthy
    function isLiquidatable(address borrower) public view returns (bool) {
        uint256 debt = debtUsdc[borrower];
        if (debt == 0) return false;
        
        uint256 collateralValue = (collateralEth[borrower] * ethPrice) / 1e18;
        uint256 thresholdValue = (collateralValue * LIQUIDATION_THRESHOLD) / 100;
        
        return debt > thresholdValue; // Debt exceeds safe threshold
    }

    // Execute liquidation
    function liquidate(address borrower, uint256 debtToRepay) public {
        require(isLiquidatable(borrower), "Position is healthy");
        require(debtToRepay > 0, "Invalid debt amount");
        require(debtToRepay <= debtUsdc[borrower], "Exceeds total debt");
        
        // Limit liquidation to 50% to prevent complete wiping in one tx (Aave style)
        uint256 maxLiquidatable = (debtUsdc[borrower] * MAX_LIQUIDATABLE_PERCENT) / 100;
        uint256 actualDebtRepaid = debtToRepay > maxLiquidatable ? maxLiquidatable : debtToRepay;
        
        // Calculate collateral to seize (with bonus)
        uint256 collateralValueUsd = (collateralEth[borrower] * ethPrice) / 1e18;
        uint256 collateralToSeizeUsd = (actualDebtRepaid * (100 + LIQUIDATION_BONUS)) / 100;
        uint256 collateralToSeizeEth = (collateralToSeizeUsd * 1e18) / ethPrice;
        
        require(collateralToSeizeEth <= collateralEth[borrower], "Not enough collateral");
        
        // Update state
        debtUsdc[borrower] -= actualDebtRepaid;
        collateralEth[borrower] -= collateralToSeizeEth;
        
        // In a real protocol:
        // 1. Liquidator transfers 'actualDebtRepaid' USDC to the protocol
        // 2. Protocol transfers 'collateralToSeizeEth' to the liquidator
        
        emit Liquidated(borrower, msg.sender, actualDebtRepaid, collateralToSeizeEth);
    }
}
```

## The Role of Liquidator Bots
- **Automation:** Humans are too slow. Liquidations are executed by automated bots (keepers) that monitor the blockchain 24/7.
- **Competition:** Multiple bots might try to liquidate the same position. The one with the highest gas price wins.
- **Profitability:** Bots only liquidate if the bonus covers their gas fees and slippage.

## Key Takeaways
- **Protocol Protection:** Liquidations ensure the lending protocol always has enough assets to cover all debts.
- **No Mercy:** Smart contracts execute liquidations automatically. No customer support can save you.
- **Liquidation Bonus:** This is the incentive that makes bots willing to pay gas fees to liquidate you.
- **Partial Liquidation:** Modern protocols (like Aave) only liquidate a portion (e.g., 50%) of the debt at a time to give the user a chance to recover.
- **Prevention:** Always keep your Health Factor above 1.5, or use stablecoins as collateral.