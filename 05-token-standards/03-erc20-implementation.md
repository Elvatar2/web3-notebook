# ERC-20 Implementation from Scratch

## Simple Definition
Building an ERC-20 token from scratch means writing all the logic (balances, transfers, approvals) yourself without relying on external libraries. This is crucial for understanding how the standard works under the hood.

## The Best Analogy
Think of this like **building a car engine from scratch**. You might not do this for a daily driver (you'd buy a certified one), but building it yourself teaches you exactly how the pistons, valves, and gears work together.

## Code Example: Minimal ERC-20

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MyCustomToken {
    string public name = "My Custom Token";
    string public symbol = "MCT";
    uint8 public decimals = 18;
    
    uint256 public totalSupply;
    
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;
    
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
    
    constructor(uint256 _initialSupply) {
        totalSupply = _initialSupply * 10 ** decimals;
        balanceOf[msg.sender] = totalSupply;
        emit Transfer(address(0), msg.sender, totalSupply);
    }
    
    function transfer(address _to, uint256 _amount) public returns (bool) {
        require(balanceOf[msg.sender] >= _amount, "Insufficient balance");
        require(_to != address(0), "Transfer to zero address");
        
        balanceOf[msg.sender] -= _amount;
        balanceOf[_to] += _amount;
        
        emit Transfer(msg.sender, _to, _amount);
        return true;
    }
    
    function approve(address _spender, uint256 _amount) public returns (bool) {
        allowance[msg.sender][_spender] = _amount;
        emit Approval(msg.sender, _spender, _amount);
        return true;
    }
    
    function transferFrom(address _from, address _to, uint256 _amount) public returns (bool) {
        require(balanceOf[_from] >= _amount, "Insufficient balance");
        require(allowance[_from][msg.sender] >= _amount, "Allowance exceeded");
        require(_to != address(0), "Transfer to zero address");
        
        balanceOf[_from] -= _amount;
        balanceOf[_to] += _amount;
        allowance[_from][msg.sender] -= _amount;
        
        emit Transfer(_from, _to, _amount);
        return true;
    }
}
```

## Key Takeaways
- **Decimals:** Tokens use 18 decimals by default (like Wei to ETH). A supply of "1000" is actually `1000 * 10**18`.
- **Zero Address Check:** Always prevent transfers to `address(0)` to avoid burning tokens accidentally.
- **Allowance Deduction:** In `transferFrom`, the allowance must be decreased by the transferred amount.
- **Events:** Emitting `Transfer` and `Approval` events is mandatory for off-chain tools to track state changes.
- **Why not use this in production?** It lacks advanced security features like reentrancy guards and overflow checks (though Solidity 0.8+ handles overflow). Use OpenZeppelin for real projects.