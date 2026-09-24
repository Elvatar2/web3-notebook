# Lending Protocol Architecture

## Simple Definition
DeFi lending protocols allow users to lend their crypto assets to earn interest, or borrow assets by providing collateral. Unlike traditional banks, these protocols are automated, permissionless, and operate 24/7 using smart contracts.

## The Best Analogy
Think of a DeFi lending protocol like a **pawn shop combined with a savings account**. You can either:
1. Deposit your gold (crypto) and earn interest from people who borrow it.
2. Bring your gold as collateral and borrow cash against it (without selling your gold).
The difference? No paperwork, no credit check, instant execution, and the "pawn shop" is a smart contract that never sleeps.

## Core Components of a Lending Protocol

```text
1. Lending Pool:
   - Holds all deposited assets
   - Calculates interest rates dynamically
   
2. Collateral System:
   - Users must over-collateralize (e.g., 150%)
   - Different assets have different LTV ratios
   
3. Interest Rate Model:
   - Rates increase as utilization increases
   - Encourages deposits when demand is high
   
4. Liquidation Engine:
   - Automatically sells collateral if value drops
   - Liquidators earn a bonus (typically 5-10%)
   
5. Oracle Integration:
   - Fetches real-time asset prices
   - Critical for accurate collateral valuation
```

## Code Example: Simplified Lending Protocol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleLendingProtocol {
    mapping(address => uint256) public deposits;
    mapping(address => uint256) public borrows;
    mapping(address => uint256) public collateral;
    
    uint256 public totalDeposits;
    uint256 public totalBorrows;
    
    uint256 public constant COLLATERAL_RATIO = 150; // 150% collateral required
    uint256 public constant LIQUIDATION_THRESHOLD = 130; // 130% triggers liquidation
    uint256 public constant LIQUIDATION_BONUS = 10; // 10% bonus for liquidators
    
    uint256 public depositRate = 5; // 5% APY for depositors
    uint256 public borrowRate = 8;  // 8% APY for borrowers
    
    event Deposited(address user, uint256 amount);
    event Borrowed(address user, uint256 amount);
    event Repaid(address user, uint256 amount);
    event Liquidated(address borrower, address liquidator, uint256 debtPaid);
    
    // Deposit ETH to earn interest
    function deposit() public payable {
        require(msg.value > 0, "Must deposit something");
        
        deposits[msg.sender] += msg.value;
        totalDeposits += msg.value;
        
        emit Deposited(msg.sender, msg.value);
    }
    
    // Borrow against collateral
    function borrow(uint256 amount) public {
        require(collateral[msg.sender] > 0, "No collateral");
        
        // Check collateral ratio
        uint256 maxBorrow = (collateral[msg.sender] * 100) / COLLATERAL_RATIO;
        require(borrows[msg.sender] + amount <= maxBorrow, "Insufficient collateral");
        
        // Check pool has enough liquidity
        uint256 availableLiquidity = totalDeposits - totalBorrows;
        require(amount <= availableLiquidity, "Pool insufficient liquidity");
        
        borrows[msg.sender] += amount;
        totalBorrows += amount;
        
        payable(msg.sender).transfer(amount);
        
        emit Borrowed(msg.sender, amount);
    }
    
    // Repay borrow
    function repay() public payable {
        require(msg.value >= borrows[msg.sender], "Insufficient repayment");
        
        totalBorrows -= borrows[msg.sender];
        borrows[msg.sender] = 0;
        
        emit Repaid(msg.sender, msg.value);
    }
    
    // Add collateral (without borrowing)
    function addCollateral() public payable {
        collateral[msg.sender] += msg.value;
    }
    
    // Liquidate undercollateralized position
    function liquidate(address borrower) public payable {
        uint256 debt = borrows[borrower];
        uint256 collateralValue = collateral[borrower];
        
        // Check if position is undercollateralized
        uint256 currentRatio = (collateralValue * 100) / debt;
        require(currentRatio < LIQUIDATION_THRESHOLD, "Position healthy");
        
        // Pay off borrower's debt
        require(msg.value >= debt, "Must pay full debt");
        
        // Calculate collateral to seize (with bonus)
        uint256 collateralToSeize = (debt * (100 + LIQUIDATION_BONUS)) / 100;
        require(collateralToSeize <= collateralValue, "Exceeds collateral");
        
        // Update state
        borrows[borrower] = 0;
        collateral[borrower] -= collateralToSeize;
        totalBorrows -= debt;
        
        // Transfer seized collateral to liquidator
        payable(msg.sender).transfer(collateralToSeize);
        
        // Refund excess ETH sent
        uint256 refund = msg.value - debt;
        if (refund > 0) {
            payable(msg.sender).transfer(refund);
        }
        
        emit Liquidated(borrower, msg.sender, debt);
    }
    
    // Check if position is healthy
    function isHealthy(address user) public view returns (bool) {
        if (borrows[user] == 0) return true;
        uint256 ratio = (collateral[user] * 100) / borrows[user];
        return ratio >= LIQUIDATION_THRESHOLD;
    }
    
    receive() external payable {}
}
```

## Interest Rate Model

```text
Utilization Rate = Total Borrows / Total Deposits

Example:
- Total Deposits: 1000 ETH
- Total Borrows: 700 ETH
- Utilization: 70%

Interest Rates (dynamic):
- Deposit APY = Borrow APY * Utilization
- If Borrow APY = 10% and Utilization = 70%
- Deposit APY = 10% * 0.7 = 7%

As utilization increases:
- Borrow rates increase exponentially
- Encourages more deposits
- Discourages excessive borrowing
```

## Key Takeaways
- **Overcollateralization:** You must deposit more than you borrow (typically 150%+).
- **Liquidation:** If collateral value drops below threshold, anyone can liquidate your position.
- **Dynamic Rates:** Interest rates adjust based on supply and demand.
- **Composability:** Borrowed assets can be used in other DeFi protocols (leverage).
- **Flash Loans:** Some protocols allow borrowing without collateral if repaid in same transaction.
- **Oracle Dependency:** Accurate price feeds are critical to prevent bad liquidations.