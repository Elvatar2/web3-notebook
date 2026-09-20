# Oracle Manipulation

## Simple Definition
Oracles are services that provide external data (like price feeds) to smart contracts. Oracle manipulation occurs when an attacker exploits the way a contract reads data from an oracle to artificially inflate or deflate prices, often to drain funds.

## The Best Analogy
Think of an oracle like a **weather station** that tells a smart contract the current temperature. If the contract uses this temperature to decide whether to turn on the heating, an attacker could manipulate the weather station (e.g., by placing it next to a heater) to trick the contract into making wrong decisions. In DeFi, this "weather station" is often a DEX price, which can be manipulated with large trades.

## Code Example: Vulnerable vs. Secure Price Feed

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface IERC20 {
    function balanceOf(address account) external view returns (uint256);
}

// VULNERABLE: Using DEX spot price as oracle
contract VulnerableLending {
    IERC20 public token;
    address public dex; // DEX contract address
    
    constructor(address _token, address _dex) {
        token = IERC20(_token);
        dex = _dex;
    }
    
    // Get price from DEX (VULNERABLE to manipulation!)
    function getTokenPrice() public view returns (uint256) {
        // This reads the current spot price from DEX
        // Attacker can manipulate this by making a large trade
        return IDex(dex).getSpotPrice(address(token));
    }
    
    function borrow(uint256 amount) public {
        uint256 price = getTokenPrice();
        // Attacker manipulates price to be very high, borrows more than they should
        require(msg.value >= (amount * 1e18) / price, "Insufficient collateral");
        // ... lending logic
    }
}

// SECURE: Using Chainlink Price Feed (decentralized oracle)
import "@chainlink/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol";

contract SecureLending {
    AggregatorV3Interface internal priceFeed;
    
    constructor() {
        // ETH/USD price feed on Ethereum mainnet
        priceFeed = AggregatorV3Interface(0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419);
    }
    
    // Get price from Chainlink (manipulation-resistant)
    function getEthPrice() public view returns (uint256) {
        (, int256 price, , , ) = priceFeed.latestRoundData();
        require(price > 0, "Invalid price");
        return uint256(price);
    }
    
    function borrow(uint256 amount) public {
        uint256 price = getEthPrice();
        require(msg.value >= (amount * 1e18) / price, "Insufficient collateral");
        // ... lending logic
    }
}
```

## Key Takeaways
- **Never Use Spot Prices:** DEX spot prices can be easily manipulated with large trades (flash loans make this even easier).
- **Use Decentralized Oracles:** Chainlink, Uniswap V3 TWAP, or other decentralized oracle networks are much harder to manipulate.
- **TWAP (Time-Weighted Average Price):** If using a DEX, use TWAP over a period (e.g., 30 minutes) instead of spot price.
- **Multiple Sources:** For critical applications, use multiple oracle sources and take the median value.
- **Flash Loan Attacks:** Oracle manipulation is often combined with flash loans to execute the attack in a single transaction.
- **Real-World Impact:** The bZx hack ($8M loss) and Cream Finance hack ($130M loss) were both oracle manipulation attacks.