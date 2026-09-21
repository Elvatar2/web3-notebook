# Secure Design Patterns

## Simple Definition
Secure design patterns are proven architectural approaches that minimize vulnerabilities in smart contracts. These patterns have been battle-tested across thousands of production contracts and form the foundation of secure Web3 development.

## The Best Analogy
Think of secure patterns like **architectural blueprints for earthquake-resistant buildings**. You wouldn't build a skyscraper without following proven engineering principles. Similarly, you shouldn't build a smart contract without following established security patterns.

## Pattern 1: Checks-Effects-Interactions

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract ChecksEffectsInteractions {
    mapping(address => uint256) public balances;
    
    // CORRECT ORDER:
    function withdraw(uint256 amount) public {
        // 1. CHECKS: Validate all conditions first
        require(balances[msg.sender] >= amount, "Insufficient balance");
        require(amount > 0, "Invalid amount");
        
        // 2. EFFECTS: Update all state variables
        balances[msg.sender] -= amount;
        
        // 3. INTERACTIONS: External calls last
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
    
    receive() external payable {}
}
```

## Pattern 2: Pull over Push

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract PullOverPush {
    mapping(address => uint256) public userBalances;
    mapping(address => uint256) public pendingWithdrawals;
    
    // BAD: Push pattern (can fail and block other operations)
    function badDistribute(address[] calldata users, uint256 amount) public {
        for (uint256 i = 0; i < users.length; i++) {
            // If one transfer fails, entire loop fails
            payable(users[i]).transfer(amount);
        }
    }
    
    // GOOD: Pull pattern (each user withdraws independently)
    function recordPayment(address user, uint256 amount) public {
        pendingWithdrawals[user] += amount;
    }
    
    function withdraw() public {
        uint256 amount = pendingWithdrawals[msg.sender];
        require(amount > 0, "Nothing to withdraw");
        
        pendingWithdrawals[msg.sender] = 0;
        
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
    
    receive() external payable {}
}
```

## Pattern 3: Emergency Stop (Circuit Breaker)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/utils/Pausable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract EmergencyStop is Pausable, Ownable {
    constructor() Ownable(msg.sender) {}
    
    function criticalOperation() public whenNotPaused {
        // This function can be paused in emergencies
        // ... important logic ...
    }
    
    function pause() public onlyOwner {
        _pause();
    }
    
    function unpause() public onlyOwner {
        _unpause();
    }
}
```

## Pattern 4: Rate Limiting

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract RateLimiter {
    mapping(address => uint256) public lastWithdrawalTime;
    uint256 public constant COOLDOWN = 1 days;
    uint256 public constant MAX_DAILY_WITHDRAWAL = 1 ether;
    mapping(address => uint256) public dailyWithdrawals;
    mapping(address => uint256) public lastResetTime;
    
    function withdraw(uint256 amount) public {
        require(amount <= MAX_DAILY_WITHDRAWAL, "Exceeds daily limit");
        
        // Reset daily counter if new day
        if (block.timestamp >= lastResetTime[msg.sender] + 1 days) {
            dailyWithdrawals[msg.sender] = 0;
            lastResetTime[msg.sender] = block.timestamp;
        }
        
        require(
            dailyWithdrawals[msg.sender] + amount <= MAX_DAILY_WITHDRAWAL,
            "Daily limit exceeded"
        );
        
        // Check cooldown
        require(
            block.timestamp >= lastWithdrawalTime[msg.sender] + COOLDOWN,
            "Cooldown period"
        );
        
        dailyWithdrawals[msg.sender] += amount;
        lastWithdrawalTime[msg.sender] = block.timestamp;
        
        payable(msg.sender).transfer(amount);
    }
    
    receive() external payable {}
}
```

## Pattern 5: Multi-Signature (Multi-Sig)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MultiSigWallet {
    address[] public owners;
    uint256 public required;
    uint256 public transactionCount;
    
    struct Transaction {
        address to;
        uint256 value;
        bool executed;
        uint256 confirmations;
    }
    
    mapping(uint256 => Transaction) public transactions;
    mapping(uint256 => mapping(address => bool)) public confirmations;
    
    modifier onlyOwner() {
        bool isOwner = false;
        for (uint256 i = 0; i < owners.length; i++) {
            if (owners[i] == msg.sender) {
                isOwner = true;
                break;
            }
        }
        require(isOwner, "Not an owner");
        _;
    }
    
    constructor(address[] memory _owners, uint256 _required) {
        require(_owners.length > 0, "Owners required");
        require(_required > 0 && _required <= _owners.length, "Invalid required");
        
        owners = _owners;
        required = _required;
    }
    
    function submitTransaction(address to, uint256 value) public onlyOwner {
        transactionCount++;
        transactions[transactionCount] = Transaction({
            to: to,
            value: value,
            executed: false,
            confirmations: 0
        });
    }
    
    function confirmTransaction(uint256 txId) public onlyOwner {
        require(!confirmations[txId][msg.sender], "Already confirmed");
        
        confirmations[txId][msg.sender] = true;
        transactions[txId].confirmations++;
        
        if (transactions[txId].confirmations >= required) {
            executeTransaction(txId);
        }
    }
    
    function executeTransaction(uint256 txId) internal {
        Transaction storage txn = transactions[txId];
        require(!txn.executed, "Already executed");
        
        txn.executed = true;
        (bool success, ) = txn.to.call{value: txn.value}("");
        require(success, "Transaction failed");
    }
    
    receive() external payable {}
}
```

## Summary of Secure Patterns

| Pattern | Purpose | When to Use |
|---------|---------|-------------|
| **Checks-Effects-Interactions** | Prevent reentrancy | Every function with external calls |
| **Pull over Push** | Avoid failed transfers | Any payment distribution |
| **Emergency Stop** | Halt operations in crisis | All production contracts |
| **Rate Limiting** | Prevent abuse | Withdrawals, minting, voting |
| **Multi-Sig** | Distributed control | Treasury, admin operations |
| **ReentrancyGuard** | Extra reentrancy protection | Functions sending ETH |
| **Access Control** | Restrict sensitive functions | All admin functions |

## Key Takeaways
- **Defense in Depth:** Use multiple patterns together for maximum security.
- **OpenZeppelin:** Most of these patterns are pre-built and audited in OpenZeppelin.
- **Gas Trade-offs:** Some patterns (like pull over push) require more user interaction but are safer.
- **Testing:** Each pattern should have dedicated tests verifying its security properties.
- **Documentation:** Clearly document which patterns you use and why in your code comments.
- **Continuous Improvement:** Security patterns evolve - stay updated with the latest research.