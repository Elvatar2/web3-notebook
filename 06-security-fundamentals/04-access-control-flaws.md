# Access Control Flaws

## Simple Definition
Access control flaws occur when a smart contract fails to properly restrict who can call sensitive functions. This allows unauthorized users to perform actions like minting tokens, withdrawing funds, or changing critical settings.

## The Best Analogy
Think of access control like a **bank vault's security system**. If the vault door can be opened by anyone who walks by (no authentication), or if the security guard falls asleep and lets anyone in (weak authentication), the bank will be robbed. Proper access control ensures only authorized personnel (contract owner/admin) can access the vault (sensitive functions).

## Code Example: Vulnerable vs. Secure

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// VULNERABLE: No access control
contract VulnerableToken {
    mapping(address => uint256) public balances;
    
    // Anyone can mint tokens!
    function mint(address to, uint256 amount) public {
        balances[to] += amount;
    }
    
    // Anyone can withdraw all funds!
    function withdraw() public {
        payable(msg.sender).transfer(address(this).balance);
    }
}

// SECURE: Proper access control with Ownable
contract SecureToken {
    mapping(address => uint256) public balances;
    address public owner;
    
    constructor() {
        owner = msg.sender;
    }
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not the owner");
        _;
    }
    
    // Only owner can mint
    function mint(address to, uint256 amount) public onlyOwner {
        balances[to] += amount;
    }
    
    // Only owner can withdraw
    function withdraw() public onlyOwner {
        payable(owner).transfer(address(this).balance);
    }
}
```

## Key Takeaways
- **Always Use Access Control:** Never leave sensitive functions public without restrictions.
- **OpenZeppelin Ownable:** Use `import "@openzeppelin/contracts/access/Ownable.sol"` for robust access control.
- **Role-Based Access Control (RBAC):** For complex systems, use OpenZeppelin's `AccessControl` to assign different roles (admin, minter, burner).
- **Test Access Control:** Always write tests to verify that non-owners cannot call restricted functions.
- **Common Mistake:** Forgetting to add `onlyOwner` to functions like `mint`, `burn`, `pause`, or `withdraw`.