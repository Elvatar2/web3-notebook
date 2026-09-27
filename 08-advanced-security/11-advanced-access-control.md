# Advanced Access Control (RBAC)

## Simple Definition
While `onlyOwner` is good for simple contracts, production systems require Role-Based Access Control (RBAC). RBAC allows you to define multiple distinct roles (e.g., Minter, Pauser, Upgrader) and assign them to different addresses or multisigs, following the principle of least privilege.

## The Best Analogy
Think of RBAC like **corporate security badges**. The CEO (Owner) has access to everything. The HR manager has access only to employee records. The IT admin can reset passwords but cannot access financial data. If the IT admin's badge is stolen, the financial data remains safe. 

## Code Example: OpenZeppelin AccessControl

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/utils/Pausable.sol";

contract AdvancedToken is AccessControl, Pausable {
    // Define custom roles using keccak256 of their string names
    bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
    bytes32 public constant PAUSER_ROLE = keccak256("PAUSER_ROLE");
    bytes32 public constant UPGRADER_ROLE = keccak256("UPGRADER_ROLE");

    constructor(address admin, address minter, address pauser) {
        // Grant the admin role to the deployer
        _grantRole(DEFAULT_ADMIN_ROLE, admin);
        
        // Grant specific roles to specific addresses
        _grantRole(MINTER_ROLE, minter);
        _grantRole(PAUSER_ROLE, pauser);
        
        // Allow the PAUSER_ROLE to pause/unpause
        _grantRole(PAUSER_ROLE, address(this)); 
    }

    // Only addresses with MINTER_ROLE can mint
    function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
        _mint(to, amount);
    }

    // Only addresses with PAUSER_ROLE can pause
    function pause() public onlyRole(PAUSER_ROLE) {
        _pause();
    }

    function unpause() public onlyRole(PAUSER_ROLE) {
        _unpause();
    }

    // Admin can revoke roles if a key is compromised
    function revokeMinter(address minter) public onlyRole(DEFAULT_ADMIN_ROLE) {
        _revokeRole(MINTER_ROLE, minter);
    }
    
    // Best practice: Renounce admin role after setup if fully decentralized
    function renounceAdmin() public onlyRole(DEFAULT_ADMIN_ROLE) {
        renounceRole(DEFAULT_ADMIN_ROLE, msg.sender);
    }
}
```

## Key Takeaways
- **Principle of Least Privilege:** Give an address only the permissions it absolutely needs to do its job.
- **Role Hierarchy:** The `DEFAULT_ADMIN_ROLE` can grant and revoke all other roles. Keep this highly secure (e.g., a 5-of-9 Multisig).
- **Compartmentalization:** If a "Minter" key is compromised, the attacker cannot pause the contract or upgrade it.
- **Self-Revocation:** Contracts can hold roles themselves (e.g., a contract can pause itself under certain conditions).
- **Always Use OpenZeppelin:** Do not write custom RBAC logic; use the battle-tested `AccessControl` contract.
- **Documentation:** Clearly document which address/multisig holds which role in your project's README.