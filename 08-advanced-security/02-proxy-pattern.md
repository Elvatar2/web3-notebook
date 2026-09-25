# The Proxy Pattern Deep Dive

## Simple Definition
The Proxy Pattern is the foundational architecture that makes upgradeable contracts possible. It uses `delegatecall` to forward user interactions from a "Proxy" contract (which holds storage) to an "Implementation" contract (which holds logic), making the implementation swappable.

## The Best Analogy
Think of the proxy pattern like a **restaurant with a rotating head chef**. The restaurant building, menu system, and customer database (Proxy/Storage) stay the same. But you can fire Chef A and hire Chef B (Implementation/Logic) without changing the restaurant's address or losing customer data. Customers still come to the same address, but the food (logic) improves.

## How delegatecall Works

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// Implementation contract (the "Chef")
contract LogicV1 {
    uint256 public value;
    
    function setValue(uint256 _value) public {
        value = _value;
    }
    
    function getVersion() public pure returns (string memory) {
        return "V1";
    }
}

// Proxy contract (the "Restaurant")
contract SimpleProxy {
    address public implementation;
    
    constructor(address _impl) {
        implementation = _impl;
    }
    
    function upgrade(address newImpl) public {
        implementation = newImpl;
    }
    
    // The magic: delegatecall
    fallback() external payable {
        address impl = implementation;
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), impl, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
}

// Usage flow:
// 1. Deploy LogicV1
// 2. Deploy SimpleProxy with LogicV1 address
// 3. Call proxy.setValue(42) -> delegatecall to LogicV1
// 4. Storage (value=42) is saved in PROXY, not LogicV1
// 5. Deploy LogicV2 with new features
// 6. Call proxy.upgrade(LogivV2)
// 7. Now proxy.setValue(100) uses LogicV2 code, but SAME storage!
```

## Critical Rule: Storage Layout Must Match

```solidity
// DANGER: Breaking storage layout on upgrade!

// LogicV1
contract LogicV1 {
    uint256 public value;    // Slot 0
    string public name;      // Slot 1
}

// LogicV2 (WRONG! - Changed order)
contract LogicV2 {
    string public name;      // Slot 0 (was uint256!)
    uint256 public value;    // Slot 1 (was string!)
    // This CORRUPTS all existing data!
}

// LogicV2 (CORRECT - Append only)
contract LogicV2Safe {
    uint256 public value;    // Slot 0 (same)
    string public name;      // Slot 1 (same)
    bool public paused;      // Slot 2 (NEW - safe to add)
}
```

## Key Takeaways
- **delegatecall Context:** Code runs from implementation, but storage/msg.sender comes from proxy.
- **Storage Layout is Immutable:** Never reorder, remove, or change types of existing variables.
- **Append-Only Rule:** Only ADD new variables at the end of storage.
- **Function Selectors:** Changing function signatures can break existing integrations.
- **Testing Upgrades:** Always test upgrades on testnet with real data before mainnet.
- **EIP-1967:** Standard storage slots for proxy implementation/admin addresses.
