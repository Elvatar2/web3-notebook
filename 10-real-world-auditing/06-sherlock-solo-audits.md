# Sherlock: Solo Audits with Insurance

## Simple Definition
Sherlock is a unique audit platform that combines competitive auditing with insurance. Auditors find bugs, and if a protocol is later hacked due to a missed bug, Sherlock's insurance fund covers the losses. This aligns auditor incentives with long-term security.

## The Best Analogy
Think of Sherlock like a **home inspection company that also sells home insurance**. The inspector (auditor) checks the house thoroughly. If they miss a critical flaw (like a faulty foundation) and the house later collapses, the insurance pays for the damage. This forces inspectors to be extremely thorough.

## How Sherlock Differs from Code4rena

```text
Code4rena:
- Multiple auditors compete
- Prize pool split among findings
- No liability if a bug is missed

Sherlock:
- Multiple auditors compete (similar model)
- Prize pool + insurance fund
- Auditors share liability if a bug is missed
- Higher payouts for Critical/High findings
- Focus on "underwritten" audits (insured)
```

## The Sherlock Audit Process

```text
1. Protocol applies for an audit and pays a fee.
2. Sherlock sets up a contest with a prize pool (e.g., $30,000).
3. Sherlock adds an insurance fund (e.g., $500,000) to cover potential hacks.
4. Auditors submit findings during the contest window.
5. Lead auditor (senior) reviews and validates findings.
6. Prizes are distributed.
7. If the protocol is hacked within the insurance period (e.g., 12 months) due to a missed bug, the insurance fund covers losses.
```

## Why Sherlock Attracts Serious Auditors

```text
1. Higher Payouts:
   - Critical findings can pay $10,000 - $50,000+
   - Insurance fund adds credibility and attracts larger protocols

2. Long-Term Incentives:
   - Auditors are motivated to find EVERY bug, not just the easy ones
   - Missing a Critical bug could mean liability for the insurance payout

3. Quality Over Quantity:
   - Sherlock emphasizes thorough, well-documented findings
   - Lead auditors provide detailed feedback on submissions

4. Reputation System:
   - Top auditors build a public track record
   - High-ranked auditors get invited to private, high-paying audits
```

## Example: Sherlock-Style Finding Report

```markdown
## [C-01] Oracle manipulation via low-liquidity Uniswap pool allows theft of all collateral

### Summary
The lending protocol uses a Uniswap V2 spot price as its oracle. An attacker can use a flash loan to manipulate the spot price, borrow maximum collateral, and default on the loan, stealing all funds.

### Vulnerability Detail
The `calculateDepositRequired()` function uses:
```solidity
uint256 price = (ethBalance * 1e18) / tokenBalance;
```
This is the instantaneous spot price, which can be manipulated with a large trade.

### Impact
Complete loss of all funds in the lending pool (~$2M TVL).

### Proof of Concept
```solidity
function test_OracleManipulation() public {
    // 1. Flash loan 10,000 ETH
    // 2. Dump ETH into Uniswap pool to crash token price
    // 3. Borrow max collateral from lending pool
    // 4. Default on loan
    // 5. Repay flash loan
    assertEq(address(lendingPool).balance, 0);
}
```

### Recommended Fix
Use Chainlink price feeds or Uniswap V3 TWAP instead of spot price.
```

## Key Takeaways
- **Insurance Model:** Sherlock's unique approach aligns auditor incentives with long-term security.
- **Higher Stakes:** Missing a bug can have financial consequences, motivating thorough audits.
- **Lead Auditor Role:** Senior auditors guide the process and ensure quality.
- **Public Track Record:** Your Sherlock profile becomes your resume for future opportunities.
- **Tax & Legal:** Sherlock payments are typically in USDC or ETH. Understand your local tax obligations.