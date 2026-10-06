# Damn Vulnerable DeFi: Backdoor

## Challenge Description
To incentivize the creation of Gnosis Safe wallets, a factory contract distributes 10 DVT tokens to each newly created wallet. There are 4 users who will receive wallets. Your goal is to drain all the DVT tokens from the 4 wallets.

**Difficulty:** ⭐⭐⭐ (Hard)

**Goal:** Drain the 40 DVT tokens (10 from each of the 4 wallets) deployed by the factory.

## Vulnerable Contracts

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@gnosis.pm/safe-contracts/contracts/GnosisSafe.sol";
import "@gnosis.pm/safe-contracts/contracts/proxies/GnosisSafeProxyFactory.sol";

contract WalletDeployer {
    GnosisSafeProxyFactory public factory;
    GnosisSafe public masterCopy;

    constructor(GnosisSafeProxyFactory _factory, GnosisSafe _masterCopy) {
        factory = _factory;
        masterCopy = _masterCopy;
    }

    function deployWallet(address[] calldata owners, uint256 threshold) external {
        // Deploys a new Safe proxy
        GnosisSafeProxy proxy = factory.createProxy(address(masterCopy));
        GnosisSafe safe = GnosisSafe(address(proxy));

        // Initializes the Safe with the provided owners
        safe.setup(owners, threshold, address(0), "", address(0), address(0), 0, address(0));

        // Distributes 10 DVT tokens to the new wallet
        // ...
    }
}
```

## Vulnerability Analysis

```text
THE BUG: The Gnosis Safe's `setup` function uses `delegatecall` to initialize the proxy.
The `setup` function allows passing a `to` address and `data` to execute an initial transaction after initialization.

If the attacker can manipulate the `masterCopy` address or the initialization data, they can execute arbitrary code in the context of the newly created Safe wallet.

In this challenge, the attacker deploys a malicious contract that, when called via `delegatecall` during the Safe's setup, sets the attacker as the owner of the Safe and drains the funds.
```

## Exploit Strategy

```text
Step 1: Deploy a malicious contract that has a function to set the attacker as the owner of the calling contract (the Safe proxy).
Step 2: Call the WalletDeployer's deployWallet function, but pass the address of your malicious contract as the `to` parameter in the setup data (or manipulate the masterCopy if the factory allows).
Step 3: The factory creates a new Safe proxy and calls `setup`.
Step 4: During `setup`, the Safe delegatecalls to your malicious contract.
Step 5: Your malicious contract executes code in the context of the Safe, setting you as the owner and transferring the 10 DVT tokens to your address.
Step 6: Repeat for all 4 wallets.
```

## Key Takeaways
- **Delegatecall Risks:** `delegatecall` executes code in the caller's context. If the called contract is malicious, it can take full control of the caller's storage and funds.
- **Gnosis Safe Initialization:** The `setup` function is powerful but dangerous if the parameters are not strictly validated.
- **Factory Patterns:** When deploying contracts via factories, ensure the initialization data cannot be manipulated by the caller.
- **Lesson Learned:** Never trust the `masterCopy` or initialization data in proxy deployments unless strictly controlled.
