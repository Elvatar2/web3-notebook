# Writing Effective Proof of Concept (PoC)

## Simple Definition
A Proof of Concept (PoC) is a working code example that demonstrates how a vulnerability can be exploited. In professional auditing, a PoC is typically a Foundry test that reproduces the bug and proves the impact.

## The Best Analogy
Think of a PoC like a **crime scene recreation in a courtroom**. You don't just tell the jury "the defendant could have stolen the money." You show them exactly how: "At 10 PM, the defendant used this key to open this safe and took $10,000." A PoC is your evidence. Without it, your finding is just a theory.

## Anatomy of a Perfect PoC

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/TargetContract.sol";

contract PoC_ExploitName is Test {
    // 1. Contract instances
    TargetContract public target;
    AttackerContract public attacker;

    // 2. Setup: Deploy contracts and configure initial state
    function setUp() public {
        target = new TargetContract();
        attacker = new AttackerContract(address(target));

        // Fund contracts with initial balances
        vm.deal(address(target), 100 ether);
        vm.deal(address(attacker), 10 ether);
    }

    // 3. Test function: Execute the exploit
    function test_ExploitName() public {
        // Record initial state
        uint256 targetBalanceBefore = address(target).balance;
        uint256 attackerBalanceBefore = address(attacker).balance;

        // Execute attack
        vm.startPrank(address(attacker));
        attacker.attack();
        vm.stopPrank();

        // 4. Assert the impact
        assertEq(address(target).balance, 0, "Target should be drained");
        assertGt(
            address(attacker).balance,
            attackerBalanceBefore,
            "Attacker should profit"
        );

        // 5. Log results for clarity
        console.log("Target balance before:", targetBalanceBefore);
        console.log("Target balance after:", address(target).balance);
        console.log("Attacker profit:", address(attacker).balance - attackerBalanceBefore);
    }
}

// 6. Attacker contract (if needed)
contract AttackerContract {
    TargetContract public target;

    constructor(address _target) {
        target = TargetContract(_target);
    }

    function attack() external payable {
        // Exploit logic here
        target.vulnerableFunction();
    }

    receive() external payable {
        // Handle incoming ETH (for reentrancy, etc.)
    }
}
```

## Best Practices for Writing PoCs

```text
1. Make It Reproducible:
   - Anyone should be able to run `forge test` and see the exploit work.
   - Don't rely on external state or specific block numbers (unless testing time-based bugs).

2. Keep It Minimal:
   - Only include code necessary to demonstrate the vulnerability.
   - Remove unrelated functions or setup steps.

3. Assert the Impact:
   - Don't just show the bug exists. Prove the financial impact.
   - Use `assertEq`, `assertGt`, `assertLt` to verify the outcome.

4. Add Console Logs:
   - Use `console.log` to show state changes during the exploit.
   - This helps reviewers understand the attack flow.

5. Name Tests Clearly:
   - `test_ReentrancyDrain()` is better than `test_Exploit1()`.
   - The name should describe what the test proves.

6. Handle Edge Cases:
   - If the exploit requires specific conditions (e.g., block.timestamp > X), set them up in `setUp()`.
   - Use Foundry cheatcodes: `vm.warp()`, `vm.roll()`, `vm.prank()`, etc.
```

## Example: PoC for Oracle Manipulation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/LendingPool.sol";
import "../src/UniswapPair.sol";

contract OracleManipulationPoC is Test {
    LendingPool public lendingPool;
    UniswapPair public uniswapPair;
    IERC20 public token;

    function setUp() public {
        // Deploy tokens and pools
        token = new MockERC20("Test Token", "TT", 18);
        uniswapPair = new UniswapPair(address(token), address(0)); // WETH

        // Add liquidity to Uniswap
        token.approve(address(uniswapPair), 1000 ether);
        uniswapPair.addLiquidity{value: 1000 ether}(1000 ether);

        // Deploy lending pool with Uniswap as oracle
        lendingPool = new LendingPool(address(token), address(uniswapPair));

        // Fund lending pool with 10,000 tokens
        token.transfer(address(lendingPool), 10000 ether);

        // Fund attacker with 10,000 tokens for manipulation
        token.transfer(address(this), 10000 ether);
    }

    function test_OracleManipulation() public {
        // Record initial state
        uint256 poolBalanceBefore = token.balanceOf(address(lendingPool));

        // Step 1: Dump tokens into Uniswap to crash price
        token.approve(address(uniswapPair), 10000 ether);
        uniswapPair.swap(token, 10000 ether, 0); // Sell all tokens

        // Step 2: Borrow from lending pool at manipulated (low) price
        lendingPool.borrow(9000 ether); // Borrow 90% of pool

        // Step 3: Assert impact
        uint256 poolBalanceAfter = token.balanceOf(address(lendingPool));
        assertLt(poolBalanceAfter, poolBalanceBefore / 10, "Pool should be mostly drained");

        console.log("Pool balance before:", poolBalanceBefore);
        console.log("Pool balance after:", poolBalanceAfter);
        console.log("Loss:", poolBalanceBefore - poolBalanceAfter);
    }
}
```

## Key Takeaways
- **PoC is Your Evidence:** Without a working PoC, your finding is just a hypothesis.
- **Reproducibility is Key:** Anyone should be able to run your test and see the same result.
- **Assert Impact:** Show the financial damage, not just the bug existence.
- **Use Foundry Cheatcodes:** `vm.prank`, `vm.warp`, `vm.deal` make complex setups easy.
- **Keep It Minimal:** Only include code necessary to demonstrate the vulnerability.
- **Name Tests Clearly:** The test name should describe what it proves.
