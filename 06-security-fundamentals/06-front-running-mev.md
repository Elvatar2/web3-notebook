# Front-Running and MEV (Maximal Extractable Value)

## Simple Definition
Front-running occurs when someone sees your pending transaction in the mempool (waiting area) and submits their own transaction with a higher gas fee to get processed first. MEV is the broader concept of extracting value by manipulating transaction ordering.

## The Best Analogy
Think of front-running like **seeing someone about to buy a rare item at an auction** and quickly placing a higher bid just before them. You didn't know the item was valuable until you saw their bid, but you jumped in front to steal the opportunity. In blockchain, "bidding higher" means paying more gas to get your transaction mined first.

## Code Example: Vulnerable DEX Swap

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// VULNERABLE: Simple swap that can be front-run
contract VulnerableDEX {
    mapping(address => uint256) public tokenBalances;
    uint256 public exchangeRate = 100; // 1 ETH = 100 tokens
    
    // User wants to swap ETH for tokens
    function swapETHForTokens() public payable {
        uint256 tokenAmount = msg.value * exchangeRate;
        tokenBalances[msg.sender] += tokenAmount;
    }
    
    // Admin can change exchange rate (VULNERABLE to front-running!)
    function setExchangeRate(uint256 newRate) public {
        exchangeRate = newRate;
    }
}

// Attack Scenario:
// 1. Admin submits: setExchangeRate(200) with 20 gwei gas
// 2. Attacker sees this in mempool
// 3. Attacker submits: swapETHForTokens() with 30 gwei gas (higher priority)
// 4. Attacker's transaction executes FIRST at old rate (100)
// 5. Admin's transaction executes SECOND, setting new rate (200)
// 6. Attacker got tokens at a discount!

// SECURE: Use commit-reveal pattern or private mempools
contract SecureDEX {
    mapping(address => uint256) public tokenBalances;
    uint256 public exchangeRate = 100;
    mapping(bytes32 => bool) public committedRates;
    
    // Step 1: Commit the new rate (hash only, no value revealed)
    function commitRate(bytes32 rateHash) public {
        committedRates[rateHash] = true;
    }
    
    // Step 2: Reveal the actual rate (must match committed hash)
    function revealRate(uint256 newRate) public {
        bytes32 rateHash = keccak256(abi.encodePacked(newRate));
        require(committedRates[rateHash], "Rate not committed");
        exchangeRate = newRate;
        delete committedRates[rateHash];
    }
    
    function swapETHForTokens() public payable {
        uint256 tokenAmount = msg.value * exchangeRate;
        tokenBalances[msg.sender] += tokenAmount;
    }
}
```

## Key Takeaways
- **Mempool Visibility:** All pending transactions are visible to everyone before they are mined.
- **Gas Price Wars:** Attackers pay higher gas to get their transactions processed first.
- **Commit-Reveal Pattern:** Hide sensitive data by committing a hash first, then revealing the actual value later.
- **Private Mempools:** Use services like Flashbots to submit transactions directly to miners, bypassing the public mempool.
- **Slippage Tolerance:** In DEXs, always set a maximum slippage to limit losses from front-running.
- **MEV Bots:** Sophisticated bots automatically scan mempools for profitable front-running opportunities.