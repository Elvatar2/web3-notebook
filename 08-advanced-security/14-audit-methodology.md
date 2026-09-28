# Professional Audit Methodology

## Simple Definition
An audit methodology is the strict, repeatable process that professional security firms (like OpenZeppelin or Trail of Bits) follow to review smart contracts. It ensures no stone is left unturned and the final report is comprehensive and actionable.

## The Best Analogy
Think of an audit methodology like a **detective investigating a crime scene**. A detective doesn't just look around randomly. They follow a strict protocol: secure the scene (scoping), gather evidence (automated tools), interview witnesses (manual review), form a hypothesis (finding a bug), and write a detailed police report (audit report).

## The 5 Phases of a Professional Audit

```text
Phase 1: Scoping & Reconnaissance (Days 1-2)
- Understand the business logic and architecture.
- Read documentation, whitepapers, and NatSpec comments.
- Run automated tools (Slither, Mythril) to get a baseline.

Phase 2: Manual Line-by-Line Review (Days 3-7)
- Read every single line of code.
- Trace data flow: Where does user input enter? Where does it affect state?
- Check access control on every sensitive function.

Phase 3: Threat Modeling & Attack Vectors (Days 8-10)
- Ask "What if?" questions.
- What if the oracle goes down? What if the user is a contract?
- Attempt to write Proof of Concept (PoC) exploits for suspected bugs.

Phase 4: Reporting & Severity Classification (Days 11-12)
- Write clear, reproducible reports for each finding.
- Classify severity: Critical, High, Medium, Low, Informational, Gas.

Phase 5: Remediation & Final Review (Days 13-14)
- Developers fix the bugs.
- Auditor reviews the fixes to ensure they are correct and didn't introduce new bugs.
```

## Example: Professional Audit Finding Format

```markdown
## [HIGH] Reentrancy in `claimRewards` allows draining of funds

**Description:**
The `claimRewards` function sends ETH to the user before updating their reward balance. An attacker can use a malicious contract to re-enter `claimRewards` and drain the contract.

**Impact:**
Complete loss of funds locked in the reward contract.

**Proof of Concept (PoC):**
```solidity
function test_ReentrancyDrain() public {
    attackerContract.attack();
    assertEq(address(vault).balance, 0);
}
```

**Recommendation:**
Apply the Checks-Effects-Interactions pattern. Update `rewardBalances[msg.sender] = 0;` BEFORE calling `payable(msg.sender).transfer(amount);`. Alternatively, use OpenZeppelin's `ReentrancyGuard`.
```

## Key Takeaways
- **Process over Tools:** A good methodology is more important than just running Slither.
- **Business Logic is Key:** Most critical bugs are not syntax errors, but flaws in the economic or business logic.
- **Clear Communication:** An audit report is useless if the developers cannot understand or reproduce the findings.
- **Severity Matters:** Focus on Critical and High issues first. Gas optimizations are nice, but security is paramount.