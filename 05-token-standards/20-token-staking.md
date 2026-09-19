# Token Staking (Earn Rewards)

## Simple Definition
Staking allows users to lock their tokens in a smart contract for a period of time in exchange for earning rewards (usually more tokens). It incentivizes holding and reduces the circulating supply.

## The Best Analogy
Think of staking like a **fixed-term bank deposit**. You put your money in the bank for a year. The bank uses your money to lend to others, and in return, they pay you an annual interest rate (APY). You can't touch the money during the year, but you earn a profit.

## Code Example: Basic Staking Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

contract SimpleStaking is ReentrancyGuard {
    IERC20 public stakingToken;
    IERC20 public rewardToken;
    
    uint256 public rewardRate = 100; // 100 reward tokens per second (simplified)
    
    mapping(address => uint256) public stakedBalance;
    mapping(address => uint256) public userRewardPerTokenPaid;
    mapping(address => uint256) public rewards;
    
    uint256 private _totalSupply;
    uint256 public lastTimeRewardApplicable;

    constructor(address _stakingToken, address _rewardToken) {
        stakingToken = IERC20(_stakingToken);
        rewardToken = IERC20(_rewardToken);
        lastTimeRewardApplicable = block.timestamp;
    }

    function stake(uint256 amount) external nonReentrant {
        require(amount > 0, "Cannot stake 0");
        _updateRewards(msg.sender);
        
        stakedBalance[msg.sender] += amount;
        _totalSupply += amount;
        
        stakingToken.transferFrom(msg.sender, address(this), amount);
    }

    function withdraw(uint256 amount) external nonReentrant {
        require(amount > 0, "Cannot withdraw 0");
        _updateRewards(msg.sender);
        
        stakedBalance[msg.sender] -= amount;
        _totalSupply -= amount;
        
        stakingToken.transfer(msg.sender, amount);
    }

    function claimReward() external nonReentrant {
        _updateRewards(msg.sender);
        
        uint256 reward = rewards[msg.sender];
        if (reward > 0) {
            rewards[msg.sender] = 0;
            rewardToken.transfer(msg.sender, reward);
        }
    }

    // Internal logic to calculate and update rewards
    function _updateRewards(address account) internal {
        lastTimeRewardApplicable = block.timestamp;
        uint256 timePassed = block.timestamp - lastTimeRewardApplicable; // Simplified logic
        rewards[account] += stakedBalance[account] * timePassed * rewardRate / 1e18;
    }
}
```

## Key Takeaways
- **Incentivization:** Staking aligns the interests of the token holders with the long-term success of the project.
- **APY/APR:** The reward rate is usually expressed as an Annual Percentage Yield.
- **Security:** Staking contracts hold large amounts of user funds. They MUST be audited and use `ReentrancyGuard`.
- **Unstaking Period:** Many projects add a "cooldown" period (e.g., 7 days) before users can withdraw, to prevent sudden liquidity drains.