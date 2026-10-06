# Damn Vulnerable DeFi: Climber

## Challenge Description
There's a secure vault protected by a Timelock contract. The Timelock requires a 1-hour delay for any transaction. The vault holds 10 million DVT tokens. Your goal is to drain the vault.

**Difficulty:** ⭐⭐⭐⭐ (Hard)

**Goal:** Bypass the Timelock and drain the 10 million DVT tokens from the vault.

## Vulnerable Contracts

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract ClimberTimelock {
    bytes32 public constant ADMIN_ROLE = keccak256("ADMIN_ROLE");
    bytes32 public constant PROPOSER_ROLE = keccak256("PROPOSER_ROLE");

    mapping(bytes32 => uint256) public operations; // id => readyAt timestamp

    function schedule(address[] calldata targets, uint256[] calldata values, bytes[] calldata dataElements, bytes32 salt) external {
        // ... schedules operation ...
    }

    function execute(address[] calldata targets, uint256[] calldata values, bytes[] calldata dataElements, bytes32 salt) external payable {
        bytes32 id = getOperationId(targets, values, dataElements, salt);
        require(isOperationReady(id), "Not ready");

        // Execute all transactions in the batch
        for (uint i = 0; i < targets.length; i++) {
            targets[i].call{value: values[i]}(dataElements[i]);
        }
    }
}

contract ClimberVault {
    ClimberTimelock public timelock;
    IERC20 public token;

    function withdraw(address recipient) external {
        require(msg.sender == address(timelock), "Only timelock");
        token.transfer(recipient, token.balanceOf(address(this)));
    }
}
```

## Vulnerability Analysis

```text
THE BUG: The Timelock's `execute` function allows executing a batch of transactions if the operation is ready.
However, the `isOperationReady` check only verifies the timestamp, NOT who scheduled it or if the delay has truly passed for all operations.

More importantly, the attacker can use a malicious target contract in the batch execution.
During the execution of the batch, the malicious contract can call back into the Timelock to grant the attacker the `ADMIN_ROLE` or `PROPOSER_ROLE`, and then immediately schedule and execute a new operation to drain the vault, all within the same transaction batch!

Wait, the actual bug in Climber is simpler: The `execute` function doesn't check if the caller is authorized. Anyone can call `execute` if the operation is ready. But how do we make it ready instantly?
The real bug: The `schedule` function sets the readyAt timestamp. But if we can manipulate the state during execution...
Actually, the classic Climber bug is that the `execute` function allows executing operations that are "ready". The attacker can schedule an operation with a target contract that, when executed, grants the attacker the admin role in the Timelock. Then, in the SAME batch, the attacker can schedule a new operation to drain the vault. Because the batch executes sequentially, the role grant happens before the drain, and the Timelock's role checks are bypassed.
```

## Exploit Strategy

```text
Step 1: Deploy a malicious contract that can interact with the Timelock.
Step 2: Schedule a batch operation with the Timelock. The batch includes:
        a. A call to the malicious contract.
        b. A call to the Timelock to grant the attacker the ADMIN_ROLE.
        c. A call to the Timelock to schedule a new operation (drain vault).
        d. A call to the Timelock to execute the new operation immediately (if the delay is bypassed or if we manipulate the timestamp).

Actually, the simplest Climber exploit:
1. The attacker schedules an operation where the target is a malicious contract.
2. The malicious contract, when called, grants the attacker the PROPOSER_ROLE and ADMIN_ROLE in the Timelock.
3. The attacker then uses these new roles to schedule and execute the vault withdrawal.
```

## Key Takeaways
- **Timelock Flaws:** Timelocks are only as secure as their role management and execution logic.
- **Batch Execution Risks:** Executing multiple transactions in a single batch can lead to state manipulation if the order is not strictly controlled.
- **Role Manipulation:** If a contract can grant roles during execution, ensure those roles cannot be abused within the same transaction.
- **Lesson Learned:** Always validate the entire state of a Timelock before and after batch execution.
