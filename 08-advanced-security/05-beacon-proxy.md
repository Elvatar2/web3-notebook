# Beacon Proxy Pattern

## Simple Definition
The Beacon Proxy pattern allows multiple proxy contracts to share a single "Beacon" contract that stores the address of the current implementation. When you upgrade the Beacon, ALL proxies pointing to it are upgraded simultaneously.

## The Best Analogy
Think of a Beacon Proxy like a **franchise business model**. Each franchise location (Proxy) operates independently with its own customers and revenue (storage). But they all follow the same operations manual (Implementation) provided by the headquarters (Beacon). When headquarters updates the manual, ALL locations instantly follow the new procedures without needing individual visits.

## Code Example: Beacon Proxy Architecture

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/proxy/beacon/IBeacon.sol";
import "@openzeppelin/contracts/proxy/beacon/UpgradeableBeacon.sol";
import "@openzeppelin/contracts/proxy/beacon/BeaconProxy.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

// 1. The Implementation Contract (shared logic)
contract SharedToken is Initializable {
    string public name;
    string public symbol;
    uint256 public totalSupply;
    mapping(address => uint256) public balances;
    
    function initialize(string memory _name, string memory _symbol) public initializer {
        name = _name;
        symbol = _symbol;
    }
    
    function mint(address to, uint256 amount) public {
        balances[to] += amount;
        totalSupply += amount;
    }
}

// 2. Deployment Script (Conceptual)
contract BeaconDeployer {
    UpgradeableBeacon public beacon;
    BeaconProxy[] public proxies;
    
    function deploySystem() public {
        // Step 1: Deploy implementation
        SharedToken implementation = new SharedToken();
        
        // Step 2: Deploy beacon with implementation address
        beacon = new UpgradeableBeacon(address(implementation), msg.sender);
        
        // Step 3: Deploy multiple proxies pointing to the same beacon
        for (uint i = 0; i < 3; i++) {
            bytes memory initData = abi.encodeWithSignature(
                "initialize(string,string)", 
                "Token", 
                "TKN"
            );
            BeaconProxy proxy = new BeaconProxy(address(beacon), initData);
            proxies.push(proxy);
        }
    }
    
    // Step 4: Upgrade ALL proxies at once by upgrading the beacon
    function upgradeAll(address newImplementation) public {
        beacon.upgradeTo(newImplementation);
        // All 3 proxies now use the new implementation!
    }
}
```

## When to Use Beacon Proxy

```text
Perfect for:
1. Multi-tenant systems (e.g., creating separate token contracts for each user)
2. NFT collections where each NFT has its own proxy but shared logic
3. SaaS-like blockchain applications
4. When you need to upgrade hundreds of contracts simultaneously

Not ideal for:
1. Single contracts (use Transparent or UUPS instead)
2. When each contract needs independent upgrade schedules
```

## Key Takeaways
- **One-to-Many Relationship:** One Beacon controls many Proxies.
- **Atomic Upgrades:** Upgrade all proxies in a single transaction.
- **Gas Efficiency:** Deploying new proxies is cheaper since they share implementation.
- **Beacon Owner:** The Beacon owner has immense power - secure it with multisig or timelock.
- **OpenZeppelin Support:** Fully supported in @openzeppelin/contracts-upgradeable.
- **Use Case Example:** OpenSea's Seaport uses beacon proxies for order fulfillment.