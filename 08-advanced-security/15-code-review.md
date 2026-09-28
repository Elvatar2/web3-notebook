# Smart Contract Code Review Best Practices

## Simple Definition
Code review is the process where developers (or auditors) systematically examine each other's code to find bugs, improve quality, and share knowledge. In Web3, code review is a critical security layer before any code reaches an auditor or production.

## The Best Analogy
Think of code review like a **peer review in a scientific journal or a newsroom**. Before an article is published, other experts read it to check for factual errors, logical fallacies, and clarity. In smart contracts, this "peer review" prevents catastrophic financial errors from going live.

## The Code Review Checklist

```text
1. Access Control:
   - Are all sensitive functions protected (onlyOwner, onlyRole)?
   - Is the owner set correctly in the constructor/initialize?

2. State Changes & External Calls:
   - Does the code follow Checks-Effects-Interactions?
   - Are external calls (transfer, call) safe from reentrancy?

3. Math & Data Types:
   - Are there any potential overflows/underflows? (Less concern in 0.8+, but check division by zero).
   - Are uint256 used appropriately instead of smaller types?

4. User Input Validation:
   - Are function arguments checked (e.g., != address(0), > 0)?
   - Is there a maximum limit to prevent abuse?

5. Events:
   - Are all critical state changes emitting events for off-chain indexing?
```

## Example: How to Write a Good Code Review Comment

```markdown
// BAD Review Comment:
"This function looks risky." (Too vague, not actionable)

// GOOD Review Comment:
"Security Risk: The `withdraw` function on line 45 updates the user's balance AFTER sending ETH. This violates the Checks-Effects-Interactions pattern and makes it vulnerable to reentrancy attacks.

Suggestion: Move line 48 (`balances[msg.sender] -= amount;`) to before line 45, or add the `nonReentrant` modifier from OpenZeppelin."
```

## Key Takeaways
- **Egoless Review:** Code review is about improving the code, not criticizing the developer.
- **Automate the Basics:** Use linters and formatters (Prettier, Solhint) so human reviewers focus on logic, not syntax.
- **Context is King:** A reviewer must understand the *intent* of the code, not just the syntax.
- **Small PRs:** Reviewing 200 lines of code is effective; reviewing 1000 lines leads to fatigue and missed bugs.
- **Require Approvals:** Enforce a rule where no code can be merged without at least one senior developer's approval.