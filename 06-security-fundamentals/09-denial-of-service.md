# Denial of Service (DoS) Attacks

## Simple Definition
A Denial of Service attack on a smart contract makes a critical function unusable, either permanently or for an extended period. This can happen through gas limit exploits, block gas limits, or locking up contract state.

## The Best Analogy
Think of DoS like **blocking the only entrance to a store with a broken truck**. No customers can enter, no business can happen, and the store is effectively closed until the truck is removed. In smart contracts, this "broken truck" could be a failed transaction that blocks all future operations.

## Code Example: DoS via Unbounded Loop

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// VULNERABLE: DoS through unbounded loop
contract VulnerableAuction {
    address public highestBidder;
    uint256 public highestBid;
    address[] public bidders; // List of all bidders
    
    function bid() public payable {
        require(msg.value > highestBid, "Bid too low");
        
        // Refund previous highest bidder
        if (highestBidder != address(0)) {
            // VULNERABILITY: If this transfer fails, the whole bid fails
            payable(highestBidder).transfer(highestBid);
        }
        
        highestBidder = msg.sender;
        highestBid = msg.value;
        bidders.push(msg.sender);
    }
    
    // VULNERABILITY: Unbounded loop - can exceed block gas limit
    function refundAll() public {
        for (uint256 i = 0; i < bidders.length; i++) {
            // If any bidder is a contract that rejects ETH, this loop gets stuck
            payable(bidders[i]).transfer(1 wei); // Simplified
        }
    }
}

// SECURE: Pull pattern to avoid DoS
contract SecureAuction {
    address public highestBidder;
    uint256 public highestBid;
    mapping(address => uint256) public pendingReturns;
    
    function bid() public payable {
        require(msg.value > highestBid, "Bid too low");
        
        // Record the refund instead of sending it
        if (highestBidder != address(0)) {
            pendingReturns[highestBidder] += highestBid;
        }
        
        highestBidder = msg.sender;
        highestBid = msg.value;
    }
    
    // Users withdraw their own refunds (no unbounded loop)
    function withdraw() public {
        uint256 amount = pendingReturns[msg.sender];
        require(amount > 0, "Nothing to withdraw");
        
        pendingReturns[msg.sender] = 0;
        payable(msg.sender).transfer(amount);
    }
}
```

## Types of DoS Attacks

```text
1. Gas Limit DoS:
   - Loop grows too large, exceeds block gas limit
   - Function becomes permanently unusable
   
2. Block Gas Limit DoS:
   - Attacker sends many small transactions to fill blocks
   - Legitimate transactions can't get through

3. Access Control DoS:
   - Attacker becomes the "owner" and refuses to transfer
   - Contract is locked forever

4. State Lock DoS:
   - A failed operation leaves the contract in a bad state
   - No future operations can succeed
```

## Key Takeaways
- **Avoid Unbounded Loops:** Never loop over arrays that can grow indefinitely.
- **Pull over Push:** Let users withdraw funds instead of pushing to them.
- **Gas Limits:** Be aware of the block gas limit (currently ~30 million gas).
- **Fallback Functions:** Contracts without payable fallback can reject ETH transfers.
- **Circuit Breakers:** Implement pause functionality for emergencies.
- **Testing:** Test with contracts that reject ETH to ensure your code handles failures gracefully.