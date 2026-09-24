# Staking and Reward Distribution

## Simple Definition
Staking in DeFi refers to locking up a specific token (often a governance or utility token) in a smart contract to earn rewards, gain voting power, or access premium features. Unlike yield farming (which often involves LP tokens), staking is usually single-sided.

## The Best Analogy
Think of staking like **buying a VIP membership at a club**. You lock up your membership card (tokens) for a year. In return, you get free drinks (rewards), a say in what music is played (governance voting), and access to exclusive rooms (premium features). You can't use the card while it's locked, but the benefits are worth it.

## Staking vs. Yield Farming

```text
| Feature          | Staking                  | Yield Farming               |
|------------------|--------------------------|-----------------------------|
| **Assets**       | Single token (e.g., AAVE)| LP Tokens (e.g., ETH-USDC)  |
| **Impermanent Loss** | None                  | Yes                         |
| **Primary Goal** | Governance, fixed yield  | Maximize high, variable APY |
| **Risk**         | Token price volatility   | IL + Smart contract risk    |
| **Complexity**   | Low                      | High                        |
```

## Code Example: Time-Locked Staking with Rewards

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

contract TokenStaking is ReentrancyGuard {
    IERC20 public stakingToken;
    IERC20 public rewardToken;
    
    uint256 public rewardRate = 1e18; // 1 reward token per second
    uint256 public lastUpdateTime;
    uint256 public rewardPerTokenStored;
    
    mapping(address => uint256) public userRewardPerTokenPaid;
    mapping(address => uint256) public rewards;
    mapping(address => uint256) public balances;
    mapping(address => uint256) public unlockTime;
    
    uint256 public totalSupply;
    uint256 public constant LOCK_PERIOD = 30 days;
    
    event Staked(address indexed user, uint256 amount);
    event Withdrawn(address indexed user, uint256 amount);
    event RewardPaid(address indexed user, uint256 reward);
    
    constructor(address _stakingToken, address _rewardToken) {
        stakingToken = IERC20(_stakingToken);
        rewardToken = IERC20(_rewardToken);
        lastUpdateTime = block.timestamp;
    }
    
    function rewardPerToken() public view returns (uint256) {
        if (totalSupply == 0) return rewardPerTokenStored;
        return rewardPerTokenStored + (((block.timestamp - lastUpdateTime) * rewardRate * 1e18) / totalSupply);
    }
    
    function earned(address account) public view returns (uint256) {
        return ((balances[account] * (rewardPerToken() - userRewardPerTokenPaid[account])) / 1e18) + rewards[account];
    }
    
    modifier updateReward(address account) {
        rewardPerTokenStored = rewardPerToken();
        lastUpdateTime = block.timestamp;
        if (account != address(0)) {
            rewards[account] = earned(account);
            userRewardPerTokenPaid[account] = rewardPerTokenStored;
        }
        _;
    }
    
    function stake(uint256 amount) public nonReentrant updateReward(msg.sender) {
        require(amount > 0, "Cannot stake 0");
        
        // If already staked, extend lock period
        if (balances[msg.sender] > 0) {
            unlockTime[msg.sender] = block.timestamp + LOCK_PERIOD;
        } else {
            unlockTime[msg.sender] = block.timestamp + LOCK_PERIOD;
        }
        
        stakingToken.transferFrom(msg.sender, address(this), amount);
        balances[msg.sender] += amount;
        totalSupply += amount;
        
        emit Staked(msg.sender, amount);
    }
    
    function withdraw(uint256 amount) public nonReentrant updateReward(msg.sender) {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        require(block.timestamp >= unlockTime[msg.sender], "Tokens are still locked");
        
        balances[msg.sender] -= amount;
        totalSupply -= amount;
        stakingToken.transfer(msg.sender, amount);
        
        emit Withdrawn(msg.sender, amount);
    }
    
    function getReward() public nonReentrant updateReward(msg.sender) {
        uint256 reward = rewards[msg.sender];
        if (reward > 0) {
            rewards[msg.sender] = 0;
            rewardToken.transfer(msg.sender, reward);
            emit RewardPaid(msg.sender, reward);
        }
    }
    
    function exit() public {
        withdraw(balances[msg.sender]);
        getReward();
    }
}
```

## Key Takeaways
- **Single-Sided:** No need to provide paired tokens, eliminating Impermanent Loss.
- **Lock-up Periods:** Many protocols require tokens to be locked for a set time to prevent dumping.
- **Governance Power:** Staked tokens often grant voting rights in DAO proposals.
- **Boosted Rewards:** Some protocols (like Curve) offer higher APY for longer lock-up periods (veToken model).
- **Slashing:** In Proof-of-Stake blockchains, misbehaving validators can have their staked tokens "slashed" (confiscated).