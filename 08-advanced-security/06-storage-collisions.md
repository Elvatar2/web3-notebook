# Storage Collisions in Upgradeable Contracts

## Simple Definition
A storage collision occurs when an upgraded implementation contract uses the same storage slots as the original, but for different variables. This corrupts existing data and can lead to catastrophic bugs or exploits.

## The Best Analogy
Think of storage slots like **numbered lockers in a gym**. Locker #1 belongs to Alice (stores her towel), Locker #2 to Bob (stores his shoes). If the gym reassigns Locker #1 to store shoes and Locker #2 to store towels, Alice opens her locker expecting a towel but finds shoes. Chaos ensues. In smart contracts, this "chaos" can mean millions of dollars lost.

## Understanding Storage Slots

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract StorageLayoutDemo {
    // Each variable occupies a "slot" (32 bytes each)
    uint256 public a;      // Slot 0
    uint256 public b;      // Slot 1
    address public c;      // Slot 2
    
    // Smaller types can share a slot (packing)
    uint128 public d;      // Slot 3 (first 16 bytes)
    uint128 public e;      // Slot 3 (last 16 bytes)
    
    // Mappings and dynamic arrays use slot as a "pointer"
    mapping(address => uint256) public balances; // Slot 4 (but data stored elsewhere via hashing)
}
```

## The Collision Disaster

```solidity
// V1 - Original Contract
contract TokenV1 {
    uint256 public totalSupply;  // Slot 0
    mapping(address => uint256) balances; // Slot 1
}

// V2 - DANGEROUS UPGRADE (Changed variable order!)
contract TokenV2_DANGEROUS {
    mapping(address => uint256) balances; // Slot 0 (WAS totalSupply!)
    uint256 public totalSupply;  // Slot 1 (WAS balances!)
    // ALL EXISTING DATA IS NOW CORRUPTED!
}

// V2 - SAFE UPGRADE (Maintained order, only appended)
contract TokenV2_SAFE {
    uint256 public totalSupply;  // Slot 0 (unchanged)
    mapping(address => uint256) balances; // Slot 1 (unchanged)
    bool public paused;  // Slot 2 (NEW - safe addition)
}
```

## Using Storage Gaps (OpenZeppelin Pattern)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract SafeUpgradeableToken is Initializable {
    uint256 public totalSupply;  // Slot 0
    mapping(address => uint256) public balances; // Slot 1
    
    // Reserve 49 slots for future use (total 50 slots reserved)
    uint256[49] private __gap;
    
    function initialize() public initializer {
        totalSupply = 1000000;
    }
    
    // V2: Can safely add variables by using the gap
    // In V2, replace part of __gap with new variables:
    // bool public paused;        // Uses first slot of __gap
    // uint256[48] private __gap; // Remaining gap
}
```

## Detecting Collisions with Slither

```bash
# Run Slither's storage layout checker
slither . --detect storage-layout

# Output example:
# StorageLayout: TokenV2 has different storage layout than TokenV1
# - Slot 0: Changed from 'uint256 totalSupply' to 'mapping balances'
# CRITICAL: This upgrade will corrupt data!
```

## Key Takeaways
- **Never Reorder Variables:** Always maintain the exact order of existing variables.
- **Never Change Types:** uint256 cannot become address or string in an upgrade.
- **Never Remove Variables:** Even if unused, keep them to preserve slot positions.
- **Use __gap Arrays:** Reserve storage slots for future additions.
- **Test on Forks:** Always test upgrades on mainnet forks with real data.
- **Automated Checks:** Use Slither and OpenZeppelin's upgrade-safe plugins to detect collisions before deployment.
- **Inheritance Order Matters:** The order of parent contracts affects storage layout.