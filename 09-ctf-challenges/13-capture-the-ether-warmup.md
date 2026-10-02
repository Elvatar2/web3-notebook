# Capture the Ether: Warmup Challenges

## Introduction to Capture the Ether
Capture the Ether is a more advanced CTF platform than Ethernaut. It focuses on mathematical exploits, creative thinking, and deeper Solidity knowledge. The challenges are organized into categories: Warmup, Lotteries, Math, and Miscellaneous.

## Challenge 1: Call Me

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract CallMeChallenge {
    bool public isComplete = false;

    function callme() public {
        isComplete = true;
    }
}
```

**Goal:** Call the `callme()` function.

**Solution:** This is the easiest challenge. Just call the function!

```solidity
// Exploit
contract CallMeExploit {
    function exploit(CallMeChallenge target) public {
        target.callme();
    }
}
```

---

## Challenge 2: Choose a Nickname

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract NicknameChallenge {
    bytes32 public nickname;
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function setNickname(bytes32 _nickname) public {
        require(msg.sender == owner, "Only owner can set nickname");
        nickname = _nickname;
    }
}
```

**Goal:** Set a nickname for the contract.

**Solution:** You are the owner (you deployed it), so just call setNickname().

```solidity
// Exploit
contract NicknameExploit {
    function exploit(NicknameChallenge target, bytes32 name) public {
        target.setNickname(name);
    }
}
```

---

## Challenge 3: Recipient

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract RecipientChallenge {
    address public recipient;
    uint256 public donation;

    function donate() public payable {
        require(msg.value > 0, "Must send ETH");
        recipient = msg.sender;
        donation = msg.value;
    }
}
```

**Goal:** Become the recipient by donating ETH.

**Solution:** Send some ETH to the donate() function.

```solidity
// Exploit
contract RecipientExploit {
    function exploit(RecipientChallenge target) public payable {
        target.donate{value: msg.value}();
    }
}
```

## Key Takeaways from Warmup
- **Warmup challenges are easy:** They're designed to get you comfortable with the platform.
- **Focus on learning:** Don't skip these - they teach you how to interact with contracts.
- **Next up:** The real challenges begin in the Lotteries category!
- **Practice:** Try solving these on the actual Capture the Ether website before looking at solutions.
- **Build confidence:** These simple challenges build your confidence for harder exploits.

## How to Play Capture the Ether
```text
1. Go to: https://capturetheether.com/
2. Connect your MetaMask wallet (use a test wallet!)
3. Choose a challenge
4. Deploy the challenge contract (the website helps you do this)
5. Write your exploit contract
6. Call your exploit to solve the challenge
7. Verify your solution on the website
```