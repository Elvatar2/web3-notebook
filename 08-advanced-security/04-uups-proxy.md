# UUPS (Universal Upgradeable Proxy Standard)

## Simple Definition
UUPS (EIP-1822) is a more gas-efficient alternative to Transparent Proxy. The key difference: upgrade logic lives in the **implementation** contract, not the proxy. This makes the proxy simpler and cheaper, but puts more security responsibility on the implementation.

## The Best Analogy
Think of UUPS like a **self-updating smartphone app**. Instead of going to the app store (proxy) to manage updates, the app itself contains the update mechanism. When a new version is available, the app downloads and installs it. This is more efficient, but if the update mechanism has a bug, you could brick your phone.

## Code Example: UUPS Implementation

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

contract UUPSToken is UUPSUpgradeable, OwnableUpgradeable {
    // Storage (append-only!)
    string public name;
    string public symbol;
    uint256 public totalSupply;
    mapping(address => uint256) public balances;
    
    // Reserve storage slots for future upgrades
    uint256[50] private __gap;
    
    function initialize(string memory _name, string memory _symbol) public initializer {
        __Ownable_init(msg.sender);
        __UUPSUpgradeable_init();
        name = _name;
        symbol = _symbol;
    }
    
    // CRITICAL: This function lives in IMPLEMENTATION, not proxy
    function _authorizeUpgrade(address newImplementation)
        internal
        override
        onlyOwner
    {
        // Only owner can authorize upgrades
        // Add additional checks here (e.g., timelock, voting)
    }
    
    function mint(address to, uint256 amount) public onlyOwner {
        balances[to] += amount;
        totalSupply += amount;
    }
    
    function transfer(address to, uint256 amount) public {
        require(balances[msg.sender] >= amount, "Insufficient");
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }
}
```

## Transparent vs. UUPS Comparison

```text
| Feature              | Transparent Proxy      | UUPS Proxy              |
|----------------------|------------------------|-------------------------|
| **Upgrade Logic**    | In Proxy               | In Implementation       |
| **Gas Cost**         | Higher (~21k more)     | Lower                   |
| **Proxy Size**       | Larger                 | Minimal                 |
| **Security Risk**    | Lower (admin isolated) | Higher (if buggy)       |
| **Complexity**       | Simpler to understand  | More complex            |
| **Adoption**         | Very High              | Growing                 |
| **Brick Risk**       | Low                    | High (if _authorizeUpgrade broken) |
```

## The "Bricking" Risk

```solidity
// DANGER: If you deploy this as V2, the proxy is BRICKED forever!
contract BrokenUUPS is UUPSUpgradeable {
    // Forgot to implement _authorizeUpgrade!
    // Or implemented it with a bug that always reverts
    
    function _authorizeUpgrade(address) internal pure override {
        revert("Upgrade permanently disabled"); // Oops!
    }
}

// SAFETY: Always test upgrades on forked mainnet first!
// SAFETY: Keep a backup of working implementation address
```

## When to Use UUPS

```text
Use UUPS when:
- Gas optimization is critical (high-frequency upgrades)
- You have a mature security team
- You want minimal proxy bytecode
- Multiple proxies share similar upgrade logic

Use Transparent when:
- You want maximum safety
- Admin operations are infrequent
- You're new to upgradeable contracts
- Following industry best practices (most auditors prefer this)
```

## Key Takeaways
- **Gas Efficient:** Saves ~21,000 gas per upgrade vs. Transparent.
- **_authorizeUpgrade:** The critical function that controls who can upgrade.
- **Bricking Risk:** A bug in upgrade authorization can permanently lock the proxy.
- **__gap Arrays:** Reserve storage slots for future variables (OpenZeppelin convention).
- **Testing is Critical:** Always test upgrades on mainnet forks before deploying.
- **Hybrid Approach:** Some protocols use Transparent for core contracts, UUPS for peripheral ones.
