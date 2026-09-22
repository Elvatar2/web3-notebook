# Formal Verification

## Simple Definition
Formal verification is a mathematical approach to proving that a smart contract's code satisfies specific properties or specifications. Unlike testing (which checks specific cases), formal verification proves correctness for ALL possible inputs and states.

## The Best Analogy
Think of formal verification like **proving a mathematical theorem**. Testing is like checking if 2+2=4, 3+3=6, and 100+100=200. Formal verification is like proving that for ALL numbers a and b, a+b=b+a. It provides absolute certainty, not just high confidence.

## Tools for Formal Verification

```text
1. Certora Prover:
   - Industry-leading commercial tool
   - Uses CVL (Certora Verification Language)
   - Used by Aave, Compound, MakerDAO
   
2. KEVM:
   - Open-source framework
   - Based on K Framework
   - Academic and research-oriented
   
3. Solc-verify:
   - Open-source
   - Integrates with Solidity compiler
   - Uses annotations in code
```

## Example: Certora Verification Language (CVL)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// Contract to verify
contract SimpleToken {
    mapping(address => uint256) public balances;
    uint256 public totalSupply;
    
    function transfer(address to, uint256 amount) public {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }
}
```

```
// CVL specification file (SimpleToken.spec)

// Define the invariant: total supply should never change
rule totalSupplyInvariant {
    // Before and after any function call
    require totalSupply == old(totalSupply);
}

// Define correctness property: transfer should preserve total balance
rule transferPreservesBalance(address sender, address receiver, uint256 amount) {
    // Precondition
    require balances[sender] >= amount;
    
    // Execute transfer
    transfer(sender, receiver, amount);
    
    // Postcondition: total balance unchanged
    assert balances[sender] + balances[receiver] == 
           old(balances[sender]) + old(balances[receiver]);
}

// Define security property: no one can have negative balance
rule noNegativeBalances(address user) {
    assert balances[user] >= 0;
}
```

## Running Formal Verification

```bash
# Using Certora (commercial, requires license)
certoraRun SimpleToken.sol:SimpleToken   --spec SimpleToken.spec   --solc solc-0.8.20   --optimistic_loop

# Output
Verifying rule: totalSupplyInvariant
  ✓ PASSED (proved for all possible inputs)

Verifying rule: transferPreservesBalance
  ✓ PASSED (proved for all possible inputs)

Verifying rule: noNegativeBalances
  ✓ PASSED (proved for all possible inputs)

All properties verified successfully!
```

## Comparison: Testing vs. Formal Verification

| Aspect | Testing | Formal Verification |
|--------|---------|---------------------|
| **Coverage** | Specific cases | ALL possible cases |
| **Confidence** | High (but not absolute) | Mathematical certainty |
| **Speed** | Fast (seconds) | Slow (minutes to hours) |
| **Cost** | Low | High (tools, expertise) |
| **Complexity** | Easy to write | Requires math/logic skills |
| **False Positives** | Rare | Can occur (needs tuning) |
| **Best For** | Quick validation | Critical invariants |

## Key Takeaways
- **Mathematical Proof:** Provides absolute certainty that properties hold for all inputs.
- **Expensive and Complex:** Requires specialized knowledge and commercial tools.
- **Critical Invariants:** Best for proving core properties (e.g., "total supply never changes").
- **Complementary:** Use alongside testing and static analysis, not as a replacement.
- **Industry Adoption:** Used by top DeFi protocols for critical components.
- **Not a Silver Bullet:** Cannot verify everything (e.g., economic attacks, oracle manipulation).
- **Future of Security:** As tools improve, formal verification will become more accessible.