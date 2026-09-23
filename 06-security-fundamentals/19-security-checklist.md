# Ultimate Smart Contract Security Checklist

## Simple Definition
A security checklist is a comprehensive, step-by-step list of verifiable items that must be checked and confirmed before deploying a smart contract to the mainnet.

## The Best Analogy
Think of a security checklist like a **pilot's pre-flight checklist**. A pilot doesn't just hop in the cockpit and take off. They systematically check fuel, flaps, engines, and instruments. If even one item fails, the flight is aborted.

## Example: Automated Checklist via GitHub Actions

```yaml
# .github/workflows/security.yml
name: Security Checklist

on: [push, pull_request]

jobs:
  security-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install Foundry
        uses: foundry-rs/foundry-toolchain@v1
        
      - name: Run Tests (Must be >90% coverage)
        run: forge test --mc CoverageTest
        
      - name: Run Slither Static Analysis
        uses: crytic/slither-action@v0.3.0
        with:
          fail-on: high
          slither-args: "--exclude naming-convention"
          
      - name: Check Contract Size
        run: forge build --sizes
```

## Manual Pre-Deployment Checklist
- [ ] All functions follow the Checks-Effects-Interactions pattern.
- [ ] No unchecked external calls (using `ReentrancyGuard`).
- [ ] Sensitive functions have `onlyOwner` or role-based modifiers.
- [ ] Math operations are safe (Solidity 0.8+ or SafeMath).
- [ ] Oracles used are decentralized (e.g., Chainlink), not spot DEX prices.
- [ ] Admin keys are stored in a secure Multi-Sig wallet (e.g., Safe), not a single EOA.
- [ ] Contract code is verified on Etherscan immediately after deployment.
- [ ] Emergency pause mechanism is tested and ready.

## Key Takeaways
- **Never Skip Steps:** Rushing deployment to "catch a market trend" is the #1 cause of hacks.
- **Automate the Checklist:** Integrate Slither and test coverage into your CI/CD pipeline (like the YAML above).
- **Peer Review:** Have another developer review your code and this checklist before deployment.
- **Sleep on It:** If you're tired or stressed, delay the deployment. Mistakes happen when rushed.