# Damn Vulnerable DeFi: Wallet Mining

## Challenge Description
A contract incentivizes the deployment of Gnosis Safe wallets to specific addresses. It pays a bounty in DVT tokens for each wallet deployed to a whitelisted address. Your goal is to drain the contract's DVT tokens by deploying wallets to addresses you control.

**Difficulty:** ⭐⭐⭐⭐ (Hard)

**Goal:** Exploit the wallet deployment mechanism to claim bounties and drain the contract.

## Vulnerable Contracts

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract WalletMining {
    GnosisSafeProxyFactory public factory;
    address public masterCopy;
    mapping(address => bool) public authorized;
    IERC20 public token;

    function init(address[] calldata users, address[] calldata wallets) external {
        // Authorizes specific wallet addresses for specific users
    }

    function deploy(address user, address wallet, uint256 salt) external {
        require(authorized[wallet], "Not authorized");

        // Deploys a Safe proxy to the specific 'wallet' address using CREATE2
        // ...

        // Pays bounty to the user
        token.transfer(user, 10 ether);
    }
}
```

## Vulnerability Analysis

```text
THE BUG: The contract uses `CREATE2` to deploy wallets to specific addresses.
The `CREATE2` address is deterministic: `keccak256(0xff ++ factoryAddress ++ salt ++ keccak256(initCode))`.

If the attacker can manipulate the `salt` or the `initCode` (by providing a malicious masterCopy or initialization data), they can deploy a wallet to an address they control, even if it's not the intended address.

Furthermore, if the `authorized` mapping check is flawed or can be bypassed, the attacker can deploy wallets to any address and claim the bounty.

In this challenge, the attacker finds a specific `salt` value that results in a wallet address they control, then calls `deploy` to claim the bounty.
```

## Exploit Strategy

```text
Step 1: Calculate the `CREATE2` address for various `salt` values until you find one that matches an address you control (or a specific target address).
Step 2: Call the `deploy` function with the calculated `salt` and the target `wallet` address.
Step 3: The factory deploys the wallet to the target address.
Step 4: The contract checks if the wallet is authorized (if the check is flawed or if you manipulated the init code to bypass it).
Step 5: The contract pays the bounty to your address.
Step 6: Repeat to drain the contract.
```

## Key Takeaways
- **CREATE2 Determinism:** `CREATE2` allows deploying contracts to predictable addresses, but this can be exploited if the salt or init code is manipulable.
- **Authorization Checks:** Always ensure that authorization checks are robust and cannot be bypassed by manipulating deployment parameters.
- **Bounty Exploits:** Similar to Free Rider, bounties for deployment can be gamed if the deployment conditions are not strictly enforced.
- **Lesson Learned:** When using `CREATE2` for deterministic deployment, ensure the salt and init code are strictly controlled and cannot be manipulated by the caller.
