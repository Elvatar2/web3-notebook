# Decentralized Exchanges (DEX)

## Simple Definition
A Decentralized Exchange (DEX) is a peer-to-peer marketplace where users can trade cryptocurrencies directly from their wallets without intermediaries. Unlike centralized exchanges (CEX) like Binance or Coinbase, DEXs don't hold your funds or require KYC.

## The Best Analogy
Think of a CEX (Binance) like a **traditional stock exchange** where a central authority matches buyers and sellers and holds all the money. Think of a DEX (Uniswap) like a **farmers market** where buyers and sellers trade directly with each other, with automated rules (smart contracts) ensuring fair trades.

## Two Types of DEX Architecture

```text
1. Order Book DEX (like dYdX, Serum):
   - Buyers place "bid" orders (I want to buy at $X)
   - Sellers place "ask" orders (I want to sell at $Y)
   - Matching engine finds overlapping orders
   - Similar to traditional stock exchanges
   
2. AMM DEX (like Uniswap, Curve):
   - No order book
   - Users trade against a liquidity pool
   - Prices determined by mathematical formula
   - Anyone can provide liquidity and earn fees
```

## Code Example: Simple Order Book DEX

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// Simplified Order Book DEX
contract OrderBookDEX {
    struct Order {
        address trader;
        uint256 price; // Price in wei per token
        uint256 amount; // Amount of tokens
        bool isActive;
    }
    
    // Buy orders: price -> list of orders
    mapping(uint256 => Order[]) public buyOrders;
    // Sell orders: price -> list of orders
    mapping(uint256 => Order[]) public sellOrders;
    
    // Place a buy order
    function placeBuyOrder(uint256 price, uint256 amount) public payable {
        require(msg.value == price * amount, "Incorrect ETH amount");
        
        buyOrders[price].push(Order({
            trader: msg.sender,
            price: price,
            amount: amount,
            isActive: true
        }));
    }
    
    // Place a sell order
    function placeSellOrder(uint256 price, uint256 amount) public {
        // In real DEX, this would require token approval
        sellOrders[price].push(Order({
            trader: msg.sender,
            price: price,
            amount: amount,
            isActive: true
        }));
    }
    
    // Match orders (simplified)
    function matchOrders(uint256 buyPrice, uint256 sellPrice) public {
        require(buyPrice >= sellPrice, "Prices don't match");
        
        // Find active buy and sell orders
        Order storage buyOrder = _getActiveOrder(buyOrders[buyPrice]);
        Order storage sellOrder = _getActiveOrder(sellOrders[sellPrice]);
        
        require(buyOrder.isActive && sellOrder.isActive, "No matching orders");
        
        // Execute trade
        uint256 tradeAmount = min(buyOrder.amount, sellOrder.amount);
        uint256 tradeValue = sellPrice * tradeAmount; // Use sell price
        
        // Transfer ETH from buyer to seller
        payable(sellOrder.trader).transfer(tradeValue);
        
        // Transfer tokens from seller to buyer (simplified)
        // In real DEX, this would use ERC20 transferFrom
        
        // Update orders
        buyOrder.amount -= tradeAmount;
        sellOrder.amount -= tradeAmount;
        
        if (buyOrder.amount == 0) buyOrder.isActive = false;
        if (sellOrder.amount == 0) sellOrder.isActive = false;
    }
    
    function _getActiveOrder(Order[] storage orders) internal view returns (Order storage) {
        for (uint256 i = 0; i < orders.length; i++) {
            if (orders[i].isActive) return orders[i];
        }
        revert("No active orders");
    }
    
    function min(uint256 a, uint256 b) internal pure returns (uint256) {
        return a < b ? a : b;
    }
}
```

## DEX vs. CEX Comparison

| Feature | CEX (Binance) | DEX (Uniswap) |
|---------|---------------|---------------|
| **Custody** | Exchange holds funds | You hold funds |
| **KYC** | Required | Not required |
| **Speed** | Fast (off-chain matching) | Slower (on-chain) |
| **Gas Fees** | None (off-chain) | Yes (on-chain) |
| **Liquidity** | High | Variable |
| **Security Risk** | Exchange hacks | Smart contract bugs |
| **Regulation** | Heavily regulated | Mostly unregulated |

## Key Takeaways
- **Self-Custody:** You always control your funds in a DEX (until the trade executes).
- **No KYC:** Trade anonymously with just a wallet address.
- **Gas Fees:** Every trade costs gas (can be expensive on Ethereum mainnet).
- **Slippage:** Large trades can move the price (especially in AMMs).
- **Impermanent Loss:** Liquidity providers can lose money if prices change significantly.
- **Composability:** DEXs can be integrated into other DeFi protocols (e.g., using Uniswap for price discovery).