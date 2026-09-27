# Timelock Controller

## Simple Definition
A Timelock is a smart contract that enforces a mandatory time delay between the proposal of a privileged action (like upgrading a contract or changing fees) and its actual execution. This gives users and the community time to react, audit the change, or exit the protocol if they disagree.

## The Best Analogy
Think of a Timelock like a **48-hour notice period for changing building rules**. The landlord can't just change the locks overnight. They must announce the change, wait 48 hours, and only then can they execute it. This gives tenants time to pack up and leave if they don't like the new rules.

## Code Example: Using OpenZeppelin TimelockController

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/governance/TimelockController.sol";

// Deployed separately, usually owned by a Multisig wallet
contract MyTimelock is TimelockController {
    // minDelay: minimum time before execution (e.g., 2 days)
    // proposers: addresses that can schedule transactions (e.g., DAO or Multisig)
    // executors: addresses that can execute scheduled transactions (e.g., anyone or specific relayers)
    // admin: address that can change the timelock settings (usually address(0) to make it immutable)
    constructor(
        uint256 minDelay,
        address[] memory proposers,
        address[] memory executors,
        address admin
    ) TimelockController(minDelay, proposers, executors, admin) {}
}
```

## How to Schedule and Execute a Transaction

```javascript
// Hardhat script to interact with Timelock
const { ethers } = require("hardhat");

async function main() {
  const timelockAddress = "0xTimelockAddress";
  const timelock = await ethers.getContractAt("TimelockController", timelockAddress);
  
  const targetContract = "0xTargetContractAddress";
  const value = 0;
  const data = ethers.utils.id("upgradeTo(address)").slice(0, 10) + 
               ethers.utils.defaultAbiCoder.encode(["address"], ["0xNewImplementation"]).slice(2);
  const predecessor = ethers.constants.HashZero;
  const salt = ethers.utils.id("my-unique-salt");
  const delay = 2 * 24 * 60 * 60; // 2 days in seconds

  // Step 1: Schedule the transaction (must be called by a proposer)
  await timelock.schedule(targetContract, value, data, predecessor, salt, delay);
  console.log("Transaction scheduled. Wait 2 days.");

  // Step 2: After 2 days, execute the transaction (can be called by anyone)
  // await timelock.execute(targetContract, value, data, predecessor, salt);
  // console.log("Transaction executed!");
}

main();
```

## Key Takeaways
- **Prevents Rug Pulls:** Malicious admins cannot instantly drain funds or change critical logic.
- **User Protection:** Gives token holders time to sell or withdraw if they disagree with a governance decision.
- **Multisig Synergy:** Best practice is to make the Timelock's "proposer" and "executor" a Multisig wallet (e.g., 3-of-5 signs the schedule, anyone can execute after the delay).
- **Minimum Delay:** Set a reasonable delay (e.g., 24-72 hours). Too short is unsafe; too long hinders rapid response to actual emergencies.
- **Emergency Pause Exception:** Critical security functions (like `pause()`) are often exempt from the timelock to allow rapid response to active hacks.