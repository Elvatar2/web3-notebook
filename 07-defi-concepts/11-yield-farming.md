# Yield Farming Mechanics

## Simple Definition
Yield farming (or liquidity mining) is the practice of locking up crypto assets in a DeFi protocol to earn rewards. These rewards usually come in the form of trading fees, interest, or newly minted governance tokens.

## The Best Analogy
Think of yield farming like **opening a franchise of a popular restaurant**. You provide the capital (liquidity) to set up the restaurant (the pool). In return, you get a share of the daily profits (trading fees) AND the parent company gives you free shares of their stock (governance tokens) to incentivize you to keep the restaurant open.

## How Yield Farming Works

```text
1. Provide Liquidity: User deposits Token A and Token B into a DEX pool.
2. Receive LP Tokens: User gets LP tokens representing their share of the pool.
3. Stake LP Tokens: User deposits these LP tokens into a "Farm" or "MasterChef" contract.
4. Earn Rewards: The Farm contract distributes reward tokens (e.g., SUSHI, CAKE) over time.
5. Compound or Sell: User can sell the rewards for profit, or compound them by adding more liquidity.
```

## Code Example: Simple Yield Farming Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// Mock interfaces for simplicity
interface IERC20 {
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
    function transfer(address to, uint256 amount) external returns (bool);
    function balanceOf(address account) external view returns (uint256);
}

contract YieldFarm {
    IERC20 public lpToken;      // The LP token users stake
    IERC20 public rewardToken;  // The token given as reward
    
    uint256 public rewardPerBlock = 10 * 1e18; // 10 tokens per block
    uint256 public lastRewardBlock;
    uint256 public accRewardPerShare; // Accumulated rewards per share (scaled by 1e12)
    
    uint256 public totalStaked;
    
    mapping(address => uint256) public stakedAmount;
    mapping(address => uint256) public rewardDebt;
    
    uint256 public constant ACC_REWARD_PRECISION = 1e12;
    
    constructor(address _lpToken, address _rewardToken) {
        lpToken = IERC20(_lpToken);
        rewardToken = IERC20(_rewardToken);
        lastRewardBlock = block.number;
    }
    
    // Update reward variables
    function updatePool() public {
        if (block.number <= lastRewardBlock) return;
        if (totalStaked == 0) {
            lastRewardBlock = block.number;
            return;
        }
        
        uint256 blocks = block.number - lastRewardBlock;
        uint256 reward = blocks * rewardPerBlock;
        
        accRewardPerShare += (reward * ACC_REWARD_PRECISION) / totalStaked;
        lastRewardBlock = block.number;
    }
    
    // Stake LP tokens
    function stake(uint256 amount) public {
        require(amount > 0, "Cannot stake 0");
        
        updatePool();
        
        // Pay pending rewards first
        if (stakedAmount[msg.sender] > 0) {
            uint256 pending = (stakedAmount[msg.sender] * accRewardPerShare) / ACC_REWARD_PRECISION - rewardDebt[msg.sender];
            if (pending > 0) {
                rewardToken.transfer(msg.sender, pending);
            }
        }
        
        // Update staked amount
        lpToken.transferFrom(msg.sender, address(this), amount);
        stakedAmount[msg.sender] += amount;
        totalStaked += amount;
        rewardDebt[msg.sender] = (stakedAmount[msg.sender] * accRewardPerShare) / ACC_REWARD_PRECISION;
    }
    
    // Withdraw LP tokens and claim rewards
    function withdraw(uint256 amount) public {
        require(stakedAmount[msg.sender] >= amount, "Insufficient staked");
        
        updatePool();
        
        // Pay pending rewards
        uint256 pending = (stakedAmount[msg.sender] * accRewardPerShare) / ACC_REWARD_PRECISION - rewardDebt[msg.sender];
        if (pending > 0) {
            rewardToken.transfer(msg.sender, pending);
        }
        
        // Update staked amount
        stakedAmount[msg.sender] -= amount;
        totalStaked -= amount;
        rewardDebt[msg.sender] = (stakedAmount[msg.sender] * accRewardPerShare) / ACC_REWARD_PRECISION;
        
        lpToken.transfer(msg.sender, amount);
    }
    
    // View pending rewards
    function pendingReward(address user) public view returns (uint256) {
        uint256 _accRewardPerShare = accRewardPerShare;
        if (block.number > lastRewardBlock && totalStaked != 0) {
            uint256 blocks = block.number - lastRewardBlock;
            uint256 reward = blocks * rewardPerBlock;
            _accRewardPerShare += (reward * ACC_REWARD_PRECISION) / totalStaked;
        }
        return (stakedAmount[user] * _accRewardPerShare) / ACC_REWARD_PRECISION - rewardDebt[user];
    }
}
```

## Key Takeaways
- **APR vs. APY:** APR is simple interest, APY includes compounding. Yield farming APYs can be extremely high (100%+) but are often unsustainable.
- **Reward Tokens:** Often, the reward token is the protocol's native governance token, which can be highly volatile.
- **Impermanent Loss Risk:** Yield farming usually requires providing liquidity, exposing you to IL. High rewards must outweigh IL.
- **Smart Contract Risk:** Farming contracts are complex and hold large amounts of value, making them prime targets for hacks.
- **Emission Schedules:** Protocols control inflation by reducing "rewardPerBlock" over time (halving events).