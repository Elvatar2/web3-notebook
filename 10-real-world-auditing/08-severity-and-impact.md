# Severity Classification in Professional Auditing

## Simple Definition
Severity classification is the process of categorizing vulnerabilities based on their potential impact and likelihood of exploitation. This helps protocols prioritize fixes and auditors focus on the most critical issues.

## The Best Analogy
Think of severity classification like a **hospital triage system**. When patients arrive, they're categorized:
- **Critical (Red):** Life-threatening, needs immediate attention (e.g., heart attack)
- **High (Orange):** Serious, needs attention soon (e.g., broken bone)
- **Medium (Yellow):** Moderate, can wait (e.g., sprained ankle)
- **Low (Green):** Minor, can be treated later (e.g., small cut)

In smart contract auditing, "life-threatening" means "can lose all user funds."

## Standard Severity Levels

```text
Critical (C):
- Direct loss of funds (theft, draining)
- Permanent freezing of funds
- Protocol insolvency
- Example: Reentrancy that drains all ETH

High (H):
- Temporary freezing of funds
- Significant loss of profitability
- Manipulation of core logic
- Example: Oracle manipulation allowing over-borrowing

Medium (M):
- Bugs that cause minor financial loss
- Degraded user experience
- Non-critical logic errors
- Example: Incorrect reward calculation (5% off)

Low (L):
- Edge cases with minimal impact
- Non-exploitable under normal conditions
- Example: Function fails only if block.timestamp is exactly X

Informational (I):
- Best practice recommendations
- Code style improvements
- Example: Use `safeTransfer` instead of `transfer`

Gas (G):
- Optimization suggestions
- Example: Use `++i` instead of `i++` in loops
```

## How to Classify Severity

```text
Ask these questions:

1. What is the financial impact?
   - Can funds be stolen? → Critical/High
   - Can funds be temporarily locked? → High/Medium
   - Is the impact < 1% of TVL? → Low

2. How easy is it to exploit?
   - Requires no special conditions? → Higher severity
   - Requires specific timing or large capital? → Lower severity

3. What is the likelihood?
   - Can any user trigger it? → Higher severity
   - Requires admin action? → Lower severity

4. What is the scope?
   - Affects all users? → Higher severity
   - Affects only one user? → Lower severity
```

## Example: Classifying Real Vulnerabilities

```text
Vulnerability 1: Reentrancy in withdraw()
- Impact: Can drain all funds
- Ease: Easy to exploit
- Likelihood: High
- Scope: All users
- Classification: **Critical**

Vulnerability 2: Missing access control on pause()
- Impact: Anyone can pause the contract
- Ease: Trivial
- Likelihood: Medium (why would someone do this?)
- Scope: All users
- Classification: **High**

Vulnerability 3: Incorrect decimal handling in reward calc
- Impact: Users get 0.1% less rewards
- Ease: N/A (automatic)
- Likelihood: Certain
- Scope: All users
- Classification: **Medium**

Vulnerability 4: Use of `transfer` instead of `safeTransfer`
- Impact: Fails for some tokens (e.g., USDT)
- Ease: N/A
- Likelihood: Low (only affects specific tokens)
- Scope: Users of affected tokens
- Classification: **Low**
```

## Common Misclassifications

```text
Mistake 1: Overestimating Low-severity issues
- "This function uses `transfer` instead of `safeTransfer`!" → Critical!!!
- Reality: Only affects non-standard tokens. Classification: Low

Mistake 2: Underestimating "theoretical" bugs
- "This reentrancy is hard to exploit, so it's Medium."
- Reality: If it can drain funds, it's Critical regardless of difficulty.

Mistake 3: Confusing Impact with Likelihood
- "This bug is unlikely to happen, so it's Low."
- Reality: If the impact is catastrophic, it's still High/Critical.
```

## Key Takeaways
- **Impact > Likelihood:** A bug that can drain $10M is Critical even if it's hard to exploit.
- **Be Consistent:** Use the same criteria for every finding.
- **Justify Your Classification:** Explain WHY you chose a specific severity in your report.
- **Learn from Judges:** Code4rena/Sherlock judges provide feedback on misclassifications.
- **When in Doubt, Go Higher:** It's better to overestimate severity than underestimate it.
- **Context Matters:** A "Medium" bug in a $10M protocol might be "High" in a $100M protocol.
