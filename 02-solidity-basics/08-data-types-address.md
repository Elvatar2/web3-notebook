# Data Types - address

## Simple Definition
The `address` type holds a 20-byte value (the size of an Ethereum address). It is used to store wallet addresses and contract addresses.

## The Best Analogy
Think of an address like a **bank account number**. Every person (EOA) and every business (Contract) on Ethereum has a unique account number. You need this number to send them money or interact with them.

## Code Example

```solidity
contract AddressExample {
    // A regular address - can receive ETH but cannot use .transfer()
    address public owner;
    
    // A payable address - can receive ETH and use .transfer()/.send()
    address payable public treasury;
    
    constructor() {
        // msg.sender is the address of whoever deployed this contract
        owner = msg.sender;
        
        // Convert regular address to payable
        treasury = payable(msg.sender);
    }
    
    function getBalance() public view returns (uint256) {
        // .balance returns the ETH balance of any address (in Wei)
        return owner.balance;
    }
}
```

## Key Takeaways
- **20 bytes:** All Ethereum addresses are exactly 20 bytes (40 hex characters, starting with 0x).
- **address vs address payable:** Regular `address` can hold an address. `address payable` can also receive ETH using .transfer() or .send().
- **.balance:** Every address has a .balance property that returns its ETH balance in Wei.
- **Common use:** Storing owners, users, contract references, and tracking who sent what.