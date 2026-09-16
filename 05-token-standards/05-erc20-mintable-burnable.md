# ERC-20 Mintable and Burnable Tokens

## Simple Definition
Minting is the process of creating new tokens and adding them to the total supply. Burning is the opposite: permanently destroying tokens, removing them from circulation, and reducing the total supply.

## The Best Analogy
Think of Minting like a **central bank printing new money** and injecting it into the economy. Think of Burning like **shredding physical cash** and throwing it in a fire. Once burned, those tokens are gone forever, which can increase the value of the remaining tokens (deflationary mechanism).

## Code Example: Mintable and Burnable Token

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract AdvancedToken is ERC20, ERC20Burnable, Ownable {
    constructor(uint256 initialSupply) ERC20("Advanced Token", "ADT") Ownable(msg.sender) {
        _mint(msg.sender, initialSupply * 10 ** decimals());
    }

    // Function to mint new tokens (only the owner can do this)
    function mint(address to, uint256 amount) public onlyOwner {
        _mint(to, amount);
    }

    // Anyone can burn their own tokens (inherited from ERC20Burnable)
    // function burn(uint256 amount) public override is already available!
    
    // Owner can burn tokens from someone else if they have allowance
    function burnFrom(address account, uint256 amount) public override onlyOwner {
        _burn(account, amount);
    }
}
```

## Key Takeaways
- **Inflationary vs. Deflationary:** Minting increases supply (inflationary). Burning decreases supply (deflationary).
- **ERC20Burnable:** OpenZeppelin provides this extension so any user can burn their own tokens.
- **Access Control:** Minting should almost always be restricted to an admin or a specific smart contract (like a vesting contract) using `onlyOwner`.
- **Tokenomics:** Many modern tokens use burning mechanisms (like BNB or SHIB) to reduce supply over time and potentially increase scarcity.