# Fuzz Testing with Echidna

## Simple Definition
Fuzz testing (fuzzing) is an automated testing technique that provides random, unexpected, or malformed inputs to a program to discover crashes, vulnerabilities, and unexpected behavior. Echidna is the most popular fuzzing tool for Solidity smart contracts.

## The Best Analogy
Think of fuzzing like **stress-testing a bridge by driving random vehicles over it**. Instead of just testing with normal cars (expected inputs), you drive trucks, buses, tanks, and even helicopters (random/unexpected inputs) to see if the bridge breaks under unusual conditions. Echidna does this for your smart contract by throwing millions of random inputs at it.

## Installation

```bash
# Install Echidna (requires Docker or native installation)
# Option 1: Docker (recommended)
docker pull ghcr.io/crytic/echidna/echidna:latest

# Option 2: Native installation (Linux/macOS)
# Download from: https://github.com/crytic/echidna/releases

# Verify installation
echidna --version
```

## Writing an Echidna Test

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// Contract to test
contract Token {
    mapping(address => uint256) public balances;
    uint256 public totalSupply;
    
    function mint(address to, uint256 amount) public {
        balances[to] += amount;
        totalSupply += amount;
    }
    
    function transfer(address to, uint256 amount) public {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }
}

// Echidna test contract
contract TestToken {
    Token public token;
    
    constructor() {
        token = new Token();
    }
    
    // Property: Total supply should always equal sum of all balances
    // Echidna will try to break this invariant
    function echidna_total_supply_invariant() public returns (bool) {
        uint256 sum = 0;
        // Note: In practice, you'd track balances differently
        // This is a simplified example
        return token.totalSupply() >= 0; // Always true (basic check)
    }
    
    // Property: No one should have more than total supply
    function echidna_balance_le_total(address user) public returns (bool) {
        return token.balances(user) <= token.totalSupply();
    }
    
    // Helper function for Echidna to call
    function mint(uint256 amount) public {
        token.mint(msg.sender, amount);
    }
    
    function transfer(address to, uint256 amount) public {
        token.transfer(to, amount);
    }
}
```

## Running Echidna

```bash
# Basic fuzzing
echidna . --contract TestToken

# Specify test contract and number of transactions
echidna . --contract TestToken --test-mode property --seq-len 100 --corpus-dir corpus

# Output results to a directory
echidna . --contract TestToken --corpus-dir echidna-output

# Stop after finding first failure
echidna . --contract TestToken --stop-on-fail
```

## Echidna Output Example

```text
$ echidna . --contract TestToken

Analyzing contract: TestToken
Running property tests...

echidna_balance_le_total: FAILED! 
  - Counterexample found:
    - Transaction 1: mint(1000)
    - Transaction 2: transfer(0x123..., 2000)
    - Result: balance > totalSupply (invariant violated)
  
  This indicates a bug in the transfer function!

Test results:
  - echidna_total_supply_invariant: PASSED
  - echidna_balance_le_total: FAILED (counterexample provided)

Coverage: 85% of reachable code
Transactions executed: 50,000
Time elapsed: 12 seconds
```

## Key Takeaways
- **Automated Bug Discovery:** Echidna can find bugs that manual testing and static analysis miss.
- **Invariant Testing:** Define properties that should ALWAYS be true, and Echidna tries to break them.
- **Random Inputs:** Tests with millions of random combinations that humans would never think of.
- **Counterexamples:** When a test fails, Echidna provides the exact sequence of transactions that caused the failure.
- **Complementary:** Use alongside static analysis (Slither) and unit tests for comprehensive coverage.
- **Learning Curve:** Writing good invariants requires deep understanding of your contract's logic.
- **Industry Standard:** Used by top protocols like Uniswap, Aave, and Compound.