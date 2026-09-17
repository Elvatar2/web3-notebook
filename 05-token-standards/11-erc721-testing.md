# Testing ERC-721 NFTs

## Simple Definition
Testing NFTs requires verifying unique token IDs, ownership tracking, metadata URIs, safe transfers (preventing locks in contracts), and minting limits.

## The Best Analogy
Think of testing NFTs like **quality control for limited edition collectibles**. You check that each item has a unique serial number, the certificate of authenticity is correct, the item can be transferred safely, and no counterfeits (duplicate IDs) exist.

## Code Example: Comprehensive NFT Test

```javascript
// test/ProductionNFT.test.js
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("ProductionNFT", function () {
  let nft;
  let owner, user1, user2;

  beforeEach(async function () {
    [owner, user1, user2] = await ethers.getSigners();
    const NFT = await ethers.getContractFactory("ProductionNFT");
    nft = await NFT.deploy();
    await nft.waitForDeployment();
  });

  describe("Deployment", function () {
    it("Should have correct name and symbol", async function () {
      expect(await nft.name()).to.equal("Production NFT");
      expect(await nft.symbol()).to.equal("PNFT");
    });

    it("Should have zero total supply initially", async function () {
      expect(await nft.totalSupply()).to.equal(0);
    });
  });

  describe("Minting", function () {
    it("Should mint NFTs with correct owner", async function () {
      await nft.safeMint(1, { value: ethers.parseEther("0.05") });
      
      expect(await nft.balanceOf(owner.address)).to.equal(1);
      expect(await nft.ownerOf(0)).to.equal(owner.address);
    });

    it("Should revert if insufficient payment", async function () {
      await expect(
        nft.safeMint(1, { value: ethers.parseEther("0.01") })
      ).to.be.revertedWith("Insufficient payment");
    });

    it("Should mint multiple NFTs in one transaction", async function () {
      await nft.safeMint(3, { value: ethers.parseEther("0.15") });
      
      expect(await nft.balanceOf(owner.address)).to.equal(3);
      expect(await nft.ownerOf(0)).to.equal(owner.address);
      expect(await nft.ownerOf(1)).to.equal(owner.address);
      expect(await nft.ownerOf(2)).to.equal(owner.address);
    });
  });

  describe("Transfers", function () {
    beforeEach(async function () {
      await nft.safeMint(1, { value: ethers.parseEther("0.05") });
    });

    it("Should transfer NFT between addresses", async function () {
      await nft.transferFrom(owner.address, user1.address, 0);
      
      expect(await nft.balanceOf(owner.address)).to.equal(0);
      expect(await nft.balanceOf(user1.address)).to.equal(1);
      expect(await nft.ownerOf(0)).to.equal(user1.address);
    });

    it("Should fail if transferring non-existent token", async function () {
      await expect(
        nft.transferFrom(owner.address, user1.address, 999)
      ).to.be.reverted;
    });
  });

  describe("Metadata", function () {
    it("Should have correct token URI after minting", async function () {
      await nft.safeMint(1, { value: ethers.parseEther("0.05") });
      const uri = await nft.tokenURI(0);
      expect(uri).to.include("ipfs://");
    });
  });
});
```

## Key Takeaways
- **ownerOf(tokenId):** Verify that each token ID is owned by the correct address.
- **balanceOf(address):** Check that the total count of NFTs per address is accurate.
- **tokenURI:** Ensure metadata URIs are set correctly after minting.
- **Safe Transfers:** OpenZeppelin's `safeMint` and `safeTransferFrom` prevent NFTs from being locked in contracts.
- **Reverts:** Test all failure cases (insufficient payment, exceeding max supply, transferring non-existent tokens).