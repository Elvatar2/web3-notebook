# Token Security and Common Vulnerabilities

## Simple Definition
Token security involves protecting your smart contract from exploits that could drain funds, allow unauthorized minting, or lock user assets. Even a small logic error can result in millions of dollars in losses.

## The Best Analogy
Think of token security like **bank vault design**. It's not enough to have a strong door (good code); you also need to ensure the guard doesn't fall asleep (access control), the alarm works (events), and thieves can't trick the system into opening the door twice (reentrancy).

## Common Vulnerabilities & Solutions

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

// GOOD: Secure Token Contract
contract SecureToken is ERC20, Ownable, ReentrancyGuard {
    
    constructor() ERC20("Secure Token", "SEC") Ownable(msg.sender) {
        _mint(msg.sender, 1000000 * 10**18);
    }

    // VULNERABILITY 1: Missing Access Control
    // BAD: function mint(address to, uint amount) public { _mint(to, amount); }
    // GOOD: Restrict to owner
    function safeMint(address to, uint256 amount) public onlyOwner {
        _mint(to, amount);
    }

    // VULNERABILITY 2: Reentrancy in custom withdrawal logic
    // If you have a function that sends ETH, always use nonReentrant
    function withdrawFunds() public onlyOwner nonReentrant {
        uint256 balance = address(this).balance;
        require(balance > 0, "No funds");
        
        // Send ETH BEFORE updating state (Checks-Effects-Interactions pattern)
        (bool success, ) = payable(owner()).call{value: balance}("");
        require(success, "Transfer failed");
    }

    // VULNERABILITY 3: Missing Zero Address Check (Handled by OZ, but good to know)
    // Always ensure you aren't minting or transferring to address(0)
}
```

## Key Takeaways
- **Use OpenZeppelin:** It protects against 99% of common vulnerabilities (overflow, zero-address, reentrancy).
- **Access Control:** Always use `onlyOwner` or Role-Based Access Control (RBAC) for sensitive functions like minting or pausing.
- **ReentrancyGuard:** Use this modifier on any function that sends ETH or interacts with external contracts.
- **Audits:** For production projects handling real value, always get a professional security audit (e.g., from CertiK or OpenZeppelin).
- **Test Edge Cases:** Test what happens when balances are zero, when max uint256 is reached, and when interacting with malicious contracts.