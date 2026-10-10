# Writing Professional Audit Reports

## Simple Definition
An audit report is the final deliverable of a smart contract audit. It documents all findings, their severity, proof of concept, and recommended fixes. A well-written report is clear, actionable, and helps the development team prioritize and fix vulnerabilities efficiently.

## The Best Analogy
Think of an audit report like a **medical diagnosis report**. A doctor doesn't just say "you're sick." They provide:
- Patient information (protocol details)
- Symptoms observed (vulnerabilities found)
- Severity assessment (Critical/High/Medium/Low)
- Treatment plan (recommended fixes)
- Follow-up instructions (remediation review)

A good report saves lives. A bad report can kill.

## Standard Audit Report Structure

```markdown
# [Protocol Name] Smart Contract Audit Report

## Executive Summary
- Audit dates: [Start] to [End]
- Commit hash: [Git commit]
- Scope: [List of contracts audited]
- Total findings: X Critical, Y High, Z Medium, W Low
- Overall risk assessment: [Brief summary]

## About the Audit
- Methodology used
- Tools employed (Slither, Foundry, etc.)
- Time spent

## Findings

### [C-01] Title of Critical Finding

**Severity:** Critical
**Impact:** [What can an attacker achieve?]
**Likelihood:** [How easy is it to exploit?]

**Description:**
[Detailed explanation of the vulnerability]

**Proof of Concept:**
```solidity
// Foundry test that reproduces the bug
function test_Exploit() public {
    // Step 1: Setup
    // Step 2: Exploit
    // Step 3: Assert impact
}
```

**Recommended Fix:**
[Specific code changes to fix the issue]

**Status:** [Acknowledged/Fixed/Rejected]

---

### [H-01] Title of High Finding
[Same structure as above]

---

## Appendix
- Gas optimization suggestions
- Informational findings
- Code style recommendations
```

## Example: Professional Finding Writeup

```markdown
### [C-01] Reentrancy in `withdraw()` Allows Complete Fund Drain

**Severity:** Critical (9.5/10)
**Impact:** An attacker can drain all ETH from the contract (~$2M TVL at time of audit).
**Likelihood:** High (requires only a simple attacker contract).

**Description:**
The `withdraw()` function sends ETH to the caller before updating the internal balance mapping. This violates the Checks-Effects-Interactions pattern and allows an attacker to re-enter the function and drain all funds.

Vulnerable code:
```solidity
function withdraw(uint256 amount) public {
    require(balances[msg.sender] >= amount, "Insufficient");

    // INTERACTION (bad: happens before effect)
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success, "Transfer failed");

    // EFFECT (too late!)
    balances[msg.sender] -= amount;
}
```

**Proof of Concept:**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/VulnerableVault.sol";

contract ReentrancyAttack is Test {
    VulnerableVault public vault;

    function setUp() public {
        vault = new VulnerableVault();
        vm.deal(address(vault), 100 ether);
        vm.deal(address(this), 1 ether);

        // Deposit 1 ETH to have a balance
        vault.deposit{value: 1 ether}();
    }

    function test_ReentrancyDrain() public {
        uint256 vaultBalanceBefore = address(vault).balance;

        // Attack: withdraw triggers reentrancy
        vault.withdraw(1 ether);

        // Vault should be drained
        assertEq(address(vault).balance, 0, "Vault not drained");
        assertGt(address(this).balance, 100 ether, "Attacker didn't profit");
    }

    receive() external payable {
        // Re-enter if vault still has ETH
        if (address(vault).balance >= 1 ether) {
            vault.withdraw(1 ether);
        }
    }
}
```

**Recommended Fix:**
Apply the Checks-Effects-Interactions pattern by updating the balance BEFORE sending ETH:

```solidity
function withdraw(uint256 amount) public {
    require(balances[msg.sender] >= amount, "Insufficient");

    // EFFECT (first)
    balances[msg.sender] -= amount;

    // INTERACTION (last)
    (bool success, ) = msg.sender.call{value: amount}("");
    require(success, "Transfer failed");
}
```

Alternatively, use OpenZeppelin's `ReentrancyGuard`:
```solidity
function withdraw(uint256 amount) public nonReentrant {
    // ... logic ...
}
```

**Status:** ✅ Fixed in commit `abc123`
```

## Key Takeaways
- **Clarity Over Cleverness:** Write for developers who will fix the bugs, not for other auditors.
- **PoC is Mandatory:** Every finding must have a reproducible Foundry test.
- **Specific Fixes:** Don't just say "fix this." Provide exact code changes.
- **Consistent Format:** Use the same structure for every finding.
- **Executive Summary Matters:** CTOs and founders often only read the first page. Make it count.
- **Track Status:** Work with the team to mark findings as Fixed/Acknowledged/Rejected.
