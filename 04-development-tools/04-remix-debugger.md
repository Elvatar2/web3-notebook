# Remix Debugger

## Simple Definition
The debugger lets you step through a transaction line-by-line, inspecting variables, memory, and storage at each moment.

## The Best Analogy
Think of it like **slow-motion replay in sports**. You rewind frame-by-frame to see exactly what went wrong.

## Code Example (Debug this failing transaction!)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// Contract with intentional bug for debugging practice
// Your task: find why withdraw sometimes fails!
contract DebugPractice {
    
    mapping(address => uint256) public balances;
    uint256 public totalDeposits;
    
    // Deposit: accepts ETH
    function deposit() public payable {
        balances[msg.sender] += msg.value; // Increase user balance
        totalDeposits += msg.value; // Increase total deposits
    }
    
    // Withdraw: this function sometimes fails - debug it!
    function withdraw(uint256 _amount) public {
        // Bug is here: why does require sometimes fail?
        require(balances[msg.sender] >= _amount, "Insufficient balance");
        
        // Wrong calculation that you should find in debugger
        uint256 fee = _amount / 10; // 10% fee
        
        // Main bug: we only subtract _amount instead of (_amount + fee)!
        balances[msg.sender] -= _amount; // Should be: balances[msg.sender] -= (_amount + fee);
        
        totalDeposits -= _amount;
        
        // Send ETH to user
        (bool success, ) = msg.sender.call{value: _amount - fee}("");
        require(success, "Transfer failed");
    }
    
    // Function to see a user's balance
    function getBalance(address _user) public view returns (uint256) {
        return balances[_user];
    }
}
```

### How to debug:
1. Deploy the contract
2. Call `deposit` with 1 ether
3. Call `withdraw` with 1000000000000000000 (1 ether)
4. Notice: transaction succeeds but balance calculation is wrong!
5. Go to **Debugger tab** (bug icon on left)
6. Select the withdraw transaction
7. Use **Step Over** (F10) to go line by line
8. Watch **Solidity Locals** panel - see how `fee` is calculated but not subtracted!
9. Find the bug: line `balances[msg.sender] -= _amount;` should include the fee.

### Debugger Controls:
- **Step Over (F10):** Go to next line
- **Step Into (F11):** Enter a function call
- **Step Out (Shift+F11):** Exit current function
- **Breakpoints:** Click next to line number to set

## Key Takeaways
- **Essential for Troubleshooting:** The #1 tool for finding why transactions fail.
- **Solidity Locals Panel:** Shows current values of all local variables.
- **Solidity State Panel:** Shows current values of state variables.
- **Gas Tracking:** See how much gas each operation costs.
- **Breakpoints:** Save time by jumping directly to suspicious lines.