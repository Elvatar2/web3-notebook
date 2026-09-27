# MEV (Maximal Extractable Value) Protection

## Simple Definition
MEV occurs when miners/validators reorder, include, or censor transactions in a block to extract profit. Common attacks include front-running (seeing your pending trade and jumping ahead of it) and sandwich attacks (buying before you and selling after you to manipulate the price). MEV protection involves design patterns that make your contract resistant to these exploits.

## The Best Analogy
Think of MEV like **insider trading at an auction**. If the auctioneer lets a shady bidder peek at your sealed bid before it's officially opened, that bidder can instantly place a bid just $1 higher than yours to win. MEV protection is like using a truly blind, cryptographically sealed bidding system where no one can see your bid until it's too late to react.

## Code Example: Commit-Reveal Pattern

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract CommitRevealAuction {
    mapping(address => bytes32) public commitments;
    mapping(address => uint256) public bids;
    
    uint256 public commitPhaseEnd;
    uint256 public revealPhaseEnd;
    
    constructor(uint256 _duration) {
        commitPhaseEnd = block.timestamp + _duration;
        revealPhaseEnd = commitPhaseEnd + _duration;
    }
    
    // Phase 1: Commit (Hide the actual bid value)
    function commitBid(bytes32 _commitment) public {
        require(block.timestamp < commitPhaseEnd, "Commit phase ended");
        commitments[msg.sender] = _commitment;
    }
    
    // Phase 2: Reveal (Show the actual value and prove it matches the commitment)
    function revealBid(uint256 _bidAmount, bytes32 _secret) public {
        require(block.timestamp >= commitPhaseEnd && block.timestamp < revealPhaseEnd, "Invalid reveal time");
        
        bytes32 expectedCommitment = keccak256(abi.encodePacked(_bidAmount, _secret));
        require(commitments[msg.sender] == expectedCommitment, "Invalid commitment");
        
        bids[msg.sender] = _bidAmount;
        delete commitments[msg.sender]; // Prevent reuse
    }
    
    // Example: To commit a bid of 5 ETH with secret "mysecret":
    // keccak256(abi.encodePacked(5000000000000000000, "mysecret"))
```

## Other MEV Protection Strategies

```text
1. Slippage Tolerance:
   - Always allow users to specify a minimum output amount (e.g., `amountOutMin` in Uniswap).
   - If a sandwich attack pushes the price beyond this limit, the transaction reverts, saving the user.

2. Private Mempools (Flashbots):
   - Bypass the public mempool entirely by sending transactions directly to builders/validators.
   - Prevents bots from even seeing the transaction until it's already in a block.

3. Batch Auctions:
   - Process all orders in a block at a single clearing price, eliminating the advantage of transaction ordering.

4. Two-Step Transactions:
   - Separate the intent (signing a message) from the execution (a relayer submits it), hiding the user's specific parameters until execution.
```

## Key Takeaways
- **Public Mempool = Danger Zone:** Everything in the public mempool is visible to predatory bots.
- **Commit-Reveal is Powerful:** The gold standard for preventing front-running in voting, auctions, and NFT mints.
- **Always Use Slippage Limits:** Never allow unlimited slippage in DEX swaps.
- **Flashbots is Your Friend:** For high-value transactions, use private RPC endpoints to avoid being sandwiched.
- **MEV is Inevitable:** You can't eliminate it entirely, but you can mitigate its impact on your users.