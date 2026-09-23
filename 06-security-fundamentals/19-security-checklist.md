# Ultimate Smart Contract Security Checklist

## Simple Definition
A security checklist is a comprehensive, step-by-step list of verifiable items that must be checked and confirmed before deploying a smart contract to the mainnet.

## The Best Analogy
Think of a security checklist like a **pilot's pre-flight checklist**. A pilot doesn't just hop in the cockpit and take off. They systematically check the fuel, flaps, engines, and instruments. If even one item fails, the flight is aborted. Your smart contract deployment should be no different.

## The Pre-Deployment Checklist

### 1. Code Quality & Logic
- [ ] All functions follow the Checks-Effects-Interactions pattern.
- [ ] No unchecked external calls (use `ReentrancyGuard`).
- [ ] All state variables are initialized correctly.
- [ ] No hardcoded addresses (use environment variables or constructors).
- [ ] Math operations are safe (using Solidity 0.8+ or SafeMath).

### 2. Access Control
- [ ] Sensitive functions (mint, burn, pause, withdraw) have `onlyOwner` or role-based modifiers.
- [ ] The `owner` is correctly set in the constructor.
- [ ] Plan for ownership transfer (use `Ownable2Step` to prevent accidental lockouts).

### 3. Economic & Business Logic
- [ ] Token decimals are standard (usually 18).
- [ ] Max supply limits are enforced.
- [ ] Fees and royalties are calculated correctly and don't exceed 100%.
- [ ] Oracles used are decentralized (e.g., Chainlink) and not spot DEX prices.

### 4. Testing & Analysis
- [ ] Unit test coverage is > 90%.
- [ ] Fuzzing tests (Echidna/Foundry) have been run for at least 1 hour.
- [ ] Static analysis (Slither) has been run and all high/critical issues resolved.
- [ ] Tested on a local fork of the mainnet (using Foundry or Hardhat).

### 5. Deployment & Operations
- [ ] Contract code is verified on Etherscan immediately after deployment.
- [ ] Admin keys are stored in a secure Multi-Sig wallet (e.g., Safe), not a single EOA.
- [ ] Emergency pause mechanism is tested and ready.
- [ ] A professional third-party audit has been completed and findings resolved.
- [ ] Monitoring tools (e.g., Tenderly, OpenZeppelin Defender) are set up to alert on unusual activity.

## Key Takeaways
- **Never Skip Steps:** Rushing deployment to "catch a market trend" is the #1 cause of hacks.
- **Automate the Checklist:** Integrate Slither and test coverage checks into your GitHub Actions CI/CD pipeline.
- **Peer Review:** Have another developer review your code and this checklist before deployment.
- **Immutability:** Remember, once deployed, you cannot change the code (unless using proxies). Double-check everything.
- **Sleep on It:** If you're tired or stressed, delay the deployment. Mistakes happen when rushed.