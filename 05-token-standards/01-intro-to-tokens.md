# Introduction to Tokens

## Simple Definition
In blockchain, a "coin" (like ETH or BTC) is the native currency of the network, used to pay for transaction fees (Gas). A "token" is a digital asset created and managed by a smart contract that runs on top of an existing blockchain (like Ethereum).

## The Best Analogy
Think of the blockchain (Ethereum) as a **shopping mall**. 
- **Coins (ETH):** The official currency of the country. You need it to pay for parking, electricity, and mall maintenance (Gas fees).
- **Tokens:** The gift cards or loyalty points issued by specific stores inside the mall. They are built on the mall's infrastructure but represent value specific to that store.

## Conceptual Code Example

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// A minimal conceptual representation of a token
contract SimpleTokenConcept {
    // The core of any token is a mapping of addresses to balances
    mapping(address => uint256) public balances;
    
    uint256 public totalSupply;
    
    constructor(uint256 _initialSupply) {
        totalSupply = _initialSupply;
        // Assign all initial supply to the contract deployer
        balances[msg.sender] = _initialSupply;
    }
    
    // Transfer value from sender to recipient
    function transfer(address _to, uint256 _amount) public {
        require(balances[msg.sender] >= _amount, "Insufficient balance");
        
        balances[msg.sender] -= _amount;
        balances[_to] += _amount;
    }
}
```

## Key Takeaways
- **Coins vs. Tokens:** Coins are native (ETH); tokens are smart contracts (USDT, SHIB).
- **Fungible vs. Non-Fungible:** Fungible tokens (like dollars) are identical and interchangeable. Non-fungible tokens (NFTs, like a specific painting) are unique.
- **Standards are Crucial:** Without standards (like ERC-20), every token would have different function names, making it impossible for wallets and exchanges to support them all.
- **Smart Contract Dependency:** Tokens rely entirely on the security and logic of their underlying smart contract.