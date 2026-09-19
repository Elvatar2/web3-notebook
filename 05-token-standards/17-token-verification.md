# Token Verification on Etherscan

## Simple Definition
Contract verification is the process of uploading your original Solidity source code to a block explorer (like Etherscan) so it can be matched against the deployed bytecode. This makes your contract's code publicly readable and interactive.

## The Best Analogy
Think of an unverified contract like a **vending machine with a metal box around it**. You can put money in and get a product out, but you have no idea what's inside or how it works. A verified contract is like a **transparent glass vending machine**. Everyone can see the exact mechanism, which builds trust and allows others to interact with it confidently.

## How to Verify (Hardhat)

```javascript
// 1. Add Etherscan API Key to hardhat.config.js
require("@nomicfoundation/hardhat-verify");

module.exports = {
  // ... other config
  etherscan: {
    apiKey: process.env.ETHERSCAN_API_KEY,
  },
};
```

```bash
# 2. Run the verification command after deployment
npx hardhat verify --network sepolia DEPLOYED_CONTRACT_ADDRESS

# Example:
# npx hardhat verify --network sepolia 0x1234567890abcdef1234567890abcdef12345678
```

## Key Takeaways
- **Trust:** Users and auditors can read your code to ensure there are no hidden malicious functions (like a backdoor to steal funds).
- **Interactivity:** Verified contracts allow users to read data and write transactions directly from the Etherscan UI without needing a custom frontend.
- **API Access:** Third-party tools (wallets, portfolio trackers) can easily read your token's data.
- **Immutable:** Once verified, the source code cannot be changed. If you deploy a new version, you must verify the new address.