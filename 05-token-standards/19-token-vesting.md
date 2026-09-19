# Token Vesting (Time-Locked Tokens)

## Simple Definition
Token vesting is a mechanism that locks tokens for a specific period and releases them gradually over time. It is commonly used for team members, advisors, and early investors to prevent them from dumping all their tokens at once and crashing the price.

## The Best Analogy
Think of vesting like a **trust fund for a young heir**. The money is given to them, but it's locked in a bank. They might have to wait 1 year before they can touch any of it (the "cliff"), and after that, they receive 10% of the total amount every month until it's all paid out.

## Code Example: Simple Linear Vesting

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract TokenVesting {
    IERC20 public token;
    address public beneficiary;
    
    uint256 public startTimestamp;
    uint256 public cliffDuration; // Time before first release
    uint256 public vestingDuration; // Total time for full release
    uint256 public totalAmount;
    
    uint256 public releasedAmount;

    constructor(
        address _token,
        address _beneficiary,
        uint256 _startTimestamp,
        uint256 _cliffDuration,
        uint256 _vestingDuration,
        uint256 _totalAmount
    ) {
        token = IERC20(_token);
        beneficiary = _beneficiary;
        startTimestamp = _startTimestamp;
        cliffDuration = _cliffDuration;
        vestingDuration = _vestingDuration;
        totalAmount = _totalAmount;
    }

    // Calculate how much should be released by now
    function vestedAmount() public view returns (uint256) {
        if (block.timestamp < startTimestamp + cliffDuration) {
            return 0; // Cliff hasn't passed
        }
        if (block.timestamp >= startTimestamp + vestingDuration) {
            return totalAmount; // Fully vested
        }
        // Linear calculation
        return (totalAmount * (block.timestamp - startTimestamp)) / vestingDuration;
    }

    // Release available tokens to the beneficiary
    function release() public {
        require(msg.sender == beneficiary, "Not beneficiary");
        
        uint256 vested = vestedAmount();
        uint256 toRelease = vested - releasedAmount;
        
        require(toRelease > 0, "Nothing to release");
        
        releasedAmount += toRelease;
        token.transfer(beneficiary, toRelease);
    }
}
```

## Key Takeaways
- **Cliff Period:** A waiting period where NO tokens are released. If the team member leaves before the cliff, they get nothing.
- **Linear Vesting:** Tokens are released smoothly over time (e.g., every second or every block).
- **Security:** The vesting contract holds the tokens, not the individual's wallet. This guarantees the schedule cannot be cheated.
- **OpenZeppelin:** For production, use OpenZeppelin's `VestingWallet` contract instead of writing your own.