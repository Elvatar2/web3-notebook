# Token Deployment to Testnet

## Simple Definition
Deploying a token to a testnet (like Sepolia) is the final rehearsal before launching on the Ethereum Mainnet. It allows you to verify that your contract works in a real blockchain environment without spending real money.

## The Best Analogy
Think of testnet deployment like a **dress rehearsal for a theater play**. The stage, lights, and costumes are all real, the actors perform exactly as they will on opening night, but there is no paying audience yet. It's the perfect time to catch any last-minute mistakes.

## Step 1: Environment Setup (.env)

```bash
# .env file
SEPOLIA_RPC_URL=https://sepolia.infura.io/v3/YOUR_INFURA_PROJECT_ID
PRIVATE_KEY=your_wallet_private_key_here
ETHERSCAN_API_KEY=your_etherscan_api_key
```

## Step 2: Deployment Script (Hardhat)

```javascript
// scripts/deploy-token.js
const hre = require("hardhat");

async function main() {
  console.log("Deploying SecureToken to Sepolia...");

  const SecureToken = await hre.ethers.getContractFactory("SecureToken");
  
  // Deploy the contract
  const token = await SecureToken.deploy();
  await token.waitForDeployment();

  const address = await token.getAddress();
  console.log("SecureToken deployed to:", address);

  // Wait for block confirmations for Etherscan verification
  console.log("Waiting for 5 block confirmations...");
  await token.deploymentTransaction().wait(5);

  // Verify on Etherscan
  console.log("Verifying contract on Etherscan...");
  try {
    await hre.run("verify:verify", {
      address: address,
      constructorArguments: [], // Add args here if your constructor takes them
    });
    console.log("Contract verified successfully!");
  } catch (error) {
    console.error("Verification failed:", error);
  }
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

## Step 3: Run the Deployment

```bash
# Ensure you have Sepolia ETH in your deployer wallet!
npx hardhat run scripts/deploy-token.js --network sepolia
```

## Key Takeaways
- **Testnet ETH:** You must get fake ETH from a Sepolia Faucet to pay for deployment gas.
- **Never commit .env:** Your private key must NEVER be pushed to GitHub.
- **Verification:** Verifying on Etherscan makes your contract readable and trustworthy for users.
- **Immutable:** Once deployed, the bytecode cannot be changed. Double-check your code before hitting deploy.
- **Next Step:** After successful testnet deployment, you are ready for Mainnet deployment (File 17/18).