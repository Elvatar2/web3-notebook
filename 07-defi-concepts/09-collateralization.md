# Collateralization and Health Factor

## Simple Definition
In DeFi lending, you cannot borrow without providing collateral. Collateralization is the process of locking up your crypto assets to secure a loan. The "Health Factor" is a metric that determines how safe your position is from being liquidated.

## The Best Analogy
Think of collateralization like **getting a mortgage for a house**. The bank doesn't give you a $500,000 loan based on trust; they require the house itself as collateral. If you stop paying, they take the house. In DeFi, the "bank" is a smart contract, and the "house" is your crypto (like ETH or WBTC).

## Key Concepts

```text
1. Loan-to-Value (LTV) Ratio:
   - The maximum percentage of your collateral's value you can borrow.
   - Example: 75% LTV means for every $100 of ETH, you can borrow up to $75.

2. Health Factor (HF):
   - A number representing the safety of your loan.
   - Formula: HF = (Collateral Value * Liquidation Threshold) / Borrowed Value
   - HF > 1.0: Position is safe.
   - HF = 1.0: Position is at the liquidation threshold.
   - HF < 1.0: Position can be liquidated by anyone.

3. Liquidation Threshold:
   - Usually lower than the max LTV (e.g., 80% LTV, but 75% liquidation threshold).
   - Provides a buffer zone before liquidation happens.
```

## Code Example: Health Factor Calculation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract HealthFactorCalculator {
    // Example prices (in real protocol, these come from Oracles like Chainlink)
    uint256 public ethPrice = 2000; // $2000 per ETH
    uint256 public usdcPrice = 1;   // $1 per USDC
    
    uint256 public constant LIQUIDATION_THRESHOLD = 75; // 75%
    uint256 public constant PRECISION = 100;

    mapping(address => uint256) public collateralEth; // in Wei
    mapping(address => uint256) public borrowedUsdc;  // in USDC units

    // Calculate the Health Factor for a user
    function getHealthFactor(address user) public view returns (uint256) {
        uint256 collateralValue = (collateralEth[user] * ethPrice) / 1e18;
        uint256 borrowValue = borrowedUsdc[user];
        
        if (borrowValue == 0) return type(uint256).max; // No debt = infinitely safe
        
        // HF = (Collateral Value * Liquidation Threshold) / Borrowed Value
        uint256 adjustedCollateral = (collateralValue * LIQUIDATION_THRESHOLD) / PRECISION;
        
        return (adjustedCollateral * 1e18) / borrowValue;
    }
    
    // Check if a user can borrow more
    function canBorrowMore(address user, uint256 additionalBorrow) public view returns (bool) {
        uint256 currentBorrow = borrowedUsdc[user];
        uint256 newBorrow = currentBorrow + additionalBorrow;
        
        uint256 collateralValue = (collateralEth[user] * ethPrice) / 1e18;
        uint256 maxBorrow = (collateralValue * 80) / 100; // Assuming 80% max LTV
        
        return newBorrow <= maxBorrow;
    }
    
    // Deposit ETH as collateral
    function depositCollateral() public payable {
        collateralEth[msg.sender] += msg.value;
    }
    
    // Borrow USDC against collateral
    function borrowUsdc(uint256 amount) public {
        require(canBorrowMore(msg.sender, amount), "Exceeds max LTV");
        borrowedUsdc[msg.sender] += amount;
        // In real protocol: transfer USDC to user
    }
}
```

## Real-World Example

```text
User deposits: 10 ETH (@ $2,000/ETH) = $20,000 collateral
Max LTV: 80% -> Max borrow = $16,000
Liquidation Threshold: 75%

Scenario A: User borrows $10,000 USDC
- Health Factor = ($20,000 * 0.75) / $10,000 = 1.5 (SAFE)

Scenario B: ETH price drops to $1,300
- New Collateral Value = 10 * $1,300 = $13,000
- New Health Factor = ($13,000 * 0.75) / $10,000 = 0.975 (UNSAFE - LIQUIDATION!)
```

## Key Takeaways
- **Overcollateralization is Mandatory:** DeFi has no credit scores, so you must lock up more value than you borrow.
- **Health Factor is Your Lifeline:** Always monitor your HF. Keep it above 1.5 for safety.
- **Volatile Collateral is Risky:** Using volatile assets (like meme coins) as collateral is dangerous due to sudden price drops.
- **Stablecoin Collateral:** Using stablecoins (USDC, DAI) as collateral is the safest way to borrow volatile assets.
- **Oracle Dependency:** Health Factor calculations rely entirely on accurate, real-time price feeds from Oracles.