# Diamond Pattern (EIP-2535)

## Simple Definition
The Diamond Pattern (or Diamond Storage) is an advanced upgradeable architecture that allows a single proxy to delegate calls to multiple implementation contracts called "Facets". Each facet handles a specific set of functions, enabling modular, scalable smart contracts that exceed Ethereum's 24KB contract size limit.

## The Best Analogy
Think of a Diamond contract like a **large corporation with specialized departments**. The CEO's office (Proxy) receives all requests. Depending on the request type, it routes to the appropriate department: HR (Facet A) handles employee issues, Finance (Facet B) handles payments, Legal (Facet C) handles contracts. Each department can be upgraded independently without affecting others.

## Code Example: Diamond Structure

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// Facet 1: Token Transfer Logic
contract TokenTransferFacet {
    function transfer(address to, uint256 amount) external {
        // Access diamond storage via LibDiamond
        AppStorage storage ds = LibDiamond.appStorage();
        require(ds.balances[msg.sender] >= amount, "Insufficient");
        ds.balances[msg.sender] -= amount;
        ds.balances[to] += amount;
    }
}

// Facet 2: Token Minting Logic
contract TokenMintFacet {
    function mint(address to, uint256 amount) external {
        AppStorage storage ds = LibDiamond.appStorage();
        require(msg.sender == ds.owner, "Not owner");
        ds.balances[to] += amount;
        ds.totalSupply += amount;
    }
}

// Diamond Storage (shared across all facets)
library LibDiamond {
    struct AppStorage {
        address owner;
        uint256 totalSupply;
        mapping(address => uint256) balances;
    }
    
    bytes32 constant DIAMOND_STORAGE_POSITION = keccak256("diamond.standard.diamond.storage");
    
    function appStorage() internal pure returns (AppStorage storage ds) {
        bytes32 position = DIAMOND_STORAGE_POSITION;
        assembly {
            ds.slot := position
        }
    }
}

// Diamond Proxy (routes calls to facets)
contract DiamondProxy {
    // facetAddress => function selectors
    mapping(bytes4 => address) public facetSelectors;
    
    fallback() external payable {
        address facet = facetSelectors[msg.sig];
        require(facet != address(0), "Function not found");
        
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), facet, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }
}
```

## Diamond vs. Single Implementation

```text
| Feature              | Single Implementation    | Diamond Pattern           |
|----------------------|--------------------------|---------------------------|
| **Max Size**         | 24KB limit               | Unlimited (multiple facets)|
| **Upgradability**    | Whole contract           | Per-function granularity  |
| **Complexity**       | Low                      | High                      |
| **Gas Cost**         | Lower                    | Higher (more delegatecalls)|
| **Best For**         | Simple contracts         | Large protocols (DeFi, NFT marketplaces)|
| **Examples**         | Most DApps               | Aavegotchi, Decentraland  |
```

## Key Takeaways
- **Bypasses 24KB Limit:** Split large contracts into multiple facets.
- **Granular Upgrades:** Upgrade individual functions without touching others.
- **Diamond Storage:** Use a fixed storage slot (via assembly) shared across all facets.
- **Complexity Trade-off:** Much harder to implement and audit than simple proxies.
- **EIP-2535 Standard:** Follows a formal standard with diamond cut functions.
- **Use When Necessary:** Only use Diamond pattern when you truly need it (large codebase or per-function upgrades).