# Bug Bounty Programs in Web3

## Simple Definition
A bug bounty program is an initiative where an organization offers financial rewards to independent security researchers (white-hat hackers) who find and responsibly disclose vulnerabilities in their smart contracts.

## The Best Analogy
Think of a bug bounty like a **"Wanted" poster with a reward**, but for good guys. Instead of waiting for a malicious hacker to steal funds, you offer a large cash prize to ethical hackers who find the flaw first and report it quietly.

## Example: Bug Bounty Proof of Concept (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/VulnerableVault.sol";

// This is the exact type of PoC you submit to Immunefi or Code4rena
contract VaultBugBountyPoC is Test {
    VulnerableVault public vault;
    address public attacker = address(0x1337);
    
    function setUp() public {
        vault = new VulnerableVault();
        // Fund the vault with 100 ETH
        vm.deal(address(vault), 100 ether);
    }
    
    function test_ExploitDrainsVault() public {
        uint256 initialAttackerBalance = attacker.balance;
        
        vm.startPrank(attacker);
        // 1. Exploit the vulnerability (e.g., reentrancy or logic flaw)
        vault.withdraw(100 ether); 
        vm.stopPrank();
        
        // 2. Assert that the attacker stole the funds
        assertEq(attacker.balance, initialAttackerBalance + 100 ether);
        assertEq(address(vault).balance, 0);
    }
}
```

## Key Takeaways
- **Lucrative Career:** Top bug bounty hunters on Immunefi make millions per year.
- **Responsibility:** Never exploit a bug on mainnet for personal gain (that's a crime).
- **Quality over Quantity:** One well-written Critical PoC (like the one above) is worth more than 100 Low-severity reports.
- **Learn by Reading:** Read past disclosed reports on Immunefi to learn how top hackers write their PoCs.
- **Major Platforms:** Immunefi, Code4rena, HackerOne, Sherlock.