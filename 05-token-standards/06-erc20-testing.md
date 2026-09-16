# Testing ERC-20 Tokens

## Simple Definition
Testing ensures your token behaves exactly as expected before you deploy it to the mainnet. You must verify that balances update correctly, transfers fail when they should, and only the owner can mint.

## The Best Analogy
Think of testing like **stress-testing a bridge before opening it to traffic**. You don't just assume it will hold; you drive heavy trucks over it, simulate strong winds, and check every bolt to ensure it won't collapse when real people (and real money) use it.

## Code Example: Hardhat Test (JavaScript)

```javascript
// test/AdvancedToken.test.js
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("AdvancedToken", function () {
  let token;
  let owner, user1, user2;

  beforeEach(async function () {
    [owner, user1, user2] = await ethers.getSigners();
    const Token = await ethers.getContractFactory("AdvancedToken");
    // Deploy with 1000 initial supply
    token = await Token.deploy(1000); 
    await token.waitForDeployment();
  });

  describe("Deployment", function () {
    it("Should assign the total supply to the owner", async function () {
      const ownerBalance = await token.balanceOf(owner.address);
      // 1000 * 10^18
      expect(ownerBalance).to.equal(ethers.parseEther("1000")); 
    });
  });

  describe("Transfers", function () {
    it("Should transfer tokens between accounts", async function () {
      // Transfer 50 tokens from owner to user1
      await token.transfer(user1.address, ethers.parseEther("50"));
      const user1Balance = await token.balanceOf(user1.address);
      expect(user1Balance).to.equal(ethers.parseEther("50"));
    });

    it("Should fail if sender doesn't have enough tokens", async function () {
      // user1 has 0 tokens, trying to send 1
      await expect(
        token.connect(user1).transfer(owner.address, ethers.parseEther("1"))
      ).to.be.revertedWith("ERC20: transfer amount exceeds balance");
    });
  });

  describe("Minting", function () {
    it("Should allow owner to mint new tokens", async function () {
      await token.mint(user2.address, ethers.parseEther("100"));
      const user2Balance = await token.balanceOf(user2.address);
      expect(user2Balance).to.equal(ethers.parseEther("100"));
    });

    it("Should reject minting from non-owner", async function () {
      await expect(
        token.connect(user1).mint(user2.address, ethers.parseEther("100"))
      ).to.be.revertedWithCustomError(token, "OwnableUnauthorizedAccount");
    });
  });
});
```

## Key Takeaways
- **beforeEach:** Deploys a fresh contract for every test to ensure isolation.
- **parseEther:** Converts human-readable numbers (like "50") to Wei (50 * 10^18).
- **revertedWith / revertedWithCustomError:** Crucial for testing that security checks (like `require`) actually work.
- **connect():** Simulates a transaction coming from a specific user (e.g., `user1`).
- **Run command:** `npx hardhat test`.