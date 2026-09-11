# Remix Deployer

## Simple Definition
The Deployer lets you deploy contracts to a blockchain and interact with their functions directly from the browser.

## The Best Analogy
Think of it like a **rocket launch control panel**. You choose the environment, load the contract, and press "Deploy" to send it live.

## Code Example (Deploy and interact with this!)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// Simple bank system with deposit and withdraw
contract BankPractice {
    
    // Contract owner (who deployed it)
    address public owner;
    
    // Balance for each user (mapping address to amount)
    mapping(address => uint256) public balances;
    
    // Constructor: sets the owner
    constructor() {
        owner = msg.sender; // Store deployer's address
    }
    
    // Deposit function: receives ETH and increases balance
    function deposit() public payable {
        // msg.value is the amount of ETH sent
        balances[msg.sender] += msg.value; // Increase sender's balance
    }
    
    // Withdraw function: zeros out user balance and sends ETH
    function withdraw(uint256 _amount) public {
        // Check if user has enough balance
        require(balances[msg.sender] >= _amount, "Insufficient balance");
        
        balances[msg.sender] -= _amount; // Decrease balance
        
        // Send ETH to user (using call - modern method)
        (bool success, ) = msg.sender.call{value: _amount}("");
        require(success, "Transfer failed");
    }
    
    // Function to see contract balance
    function getContractBalance() public view returns (uint256) {
        return address(this).balance; // Balance of the contract itself
    }
}
```

### How to deploy and test:
1. Compile the contract
2. In "Deploy & Run" tab, select **Remix VM** environment
3. Click **Deploy**
4. Under "Deployed Contracts":
   - Try `deposit` with value: 1 ether (enter 1 in Value field, select "ether")
   - Check `balances` with your account address
   - Try `withdraw` with amount: 500000000000000000 (0.5 ether in Wei)
   - Check `getContractBalance` before and after

## Key Takeaways
- **Remix VM:** 15 fake accounts with 100 ETH each. Perfect for testing.
- **Injected Provider:** Uses MetaMask. Deploys to real/test networks.
- **Blue Buttons:** Read-only functions (view/pure). No gas cost.
- **Orange Buttons:** State-changing functions. Cost gas.
- **Red Buttons:** Payable functions. Can receive ETH.