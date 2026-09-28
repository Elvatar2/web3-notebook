# Advanced Formal Verification (Certora & Halmos)

## Simple Definition
Advanced formal verification uses mathematical logic to prove that a smart contract's code strictly adheres to a set of rules (specifications) under ALL possible conditions. Tools like Certora (using CVL) or Halmos (symbolic testing) are used by top DeFi protocols to guarantee safety.

## The Best Analogy
Think of standard testing like **checking if a bridge holds 10 trucks**. Formal verification is like **mathematically proving the bridge's physics equations** so you know it will hold ANY number of trucks, in any weather, forever. It provides absolute certainty, not just high confidence.

## Code Example: Certora Verification Language (CVL)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleVault {
    mapping(address => uint256) public balances;
    uint256 public totalDeposits;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
        totalDeposits += msg.value;
    }

    function withdraw(uint256 amount) public {
        require(balances[msg.sender] >= amount, "Insufficient");
        balances[msg.sender] -= amount;
        totalDeposits -= amount;
        payable(msg.sender).transfer(amount);
    }
}
```

```java
// SimpleVault.spec (Certora CVL)

// Rule 1: The total deposits must always equal the sum of all user balances
// (This is an invariant that must ALWAYS be true)
rule invariantTotalDeposits {
    // For all possible addresses, sum their balances
    uint256 sum = 0;
    // (In real CVL, you use quantifiers like forall)
    assert totalDeposits == sum;
}

// Rule 2: A user's balance can never be negative
rule userBalanceNonNegative(address user) {
    assert balances[user] >= 0;
}

// Rule 3: The contract should never hold more ETH than totalDeposits
rule contractSolvency {
    assert address(this).balance == totalDeposits;
}
```

## Halmos: Symbolic Testing for Foundry

```bash
# Halmos integrates with Foundry to run symbolic tests
# Install Halmos
pip3 install halmos

# Run symbolic tests on your Foundry project
halmos --contract TestVault
```

```solidity
// Foundry test that Halmos can run symbolically
function test_WithdrawNeverExceedsBalance(uint256 depositAmount, uint256 withdrawAmount) public {
    // Halmos will try EVERY possible combination of these uint256 numbers
    vm.prank(user);
    vault.deposit{value: depositAmount}();

    vm.prank(user);
    try vault.withdraw(withdrawAmount) {
        // If it succeeds, withdrawAmount MUST be <= depositAmount
        assert(withdrawAmount <= depositAmount);
    } catch {
        // If it reverts, it's fine (e.g., Insufficient balance)
    }
}
```

## Key Takeaways
- **Absolute Certainty:** Proves correctness for infinite inputs, unlike fuzzing which tests thousands.
- **High Effort:** Writing formal specifications requires deep mathematical and logical skills.
- **Critical for DeFi:** Used by Aave, MakerDAO, and Compound for their core lending math.
- **Not a Silver Bullet:** It only proves the code matches the spec. If the spec itself is flawed, the proof is useless.
- **Future Standard:** As tools like Halmos become easier to use, formal verification will become standard for all serious DeFi protocols.