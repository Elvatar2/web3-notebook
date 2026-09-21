# Signature Replay and Phishing Attacks

## Simple Definition
Signature replay attacks occur when a valid cryptographic signature is reused (replayed) multiple times to perform unauthorized actions. Phishing attacks trick users into signing malicious messages that appear legitimate.

## The Best Analogy
Think of signature replay like **photocopying a signed check**. If you sign a check for $100, someone could photocopy it and try to cash it multiple times. A secure system needs to mark each check as "used" after the first cashing. In smart contracts, this means tracking which signatures have already been used.

## Code Example: Vulnerable vs. Secure Signature Verification

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// VULNERABLE: Signature can be replayed
contract VulnerableSignature {
    mapping(address => uint256) public balances;
    address public owner;
    
    constructor() {
        owner = msg.sender;
    }
    
    // Owner signs a message authorizing a transfer
    function verifyTransfer(
        address recipient,
        uint256 amount,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) public {
        bytes32 messageHash = keccak256(abi.encodePacked(recipient, amount));
        bytes32 ethSignedHash = toEthSignedMessageHash(messageHash);
        address signer = recoverSigner(ethSignedHash, v, r, s);
        
        require(signer == owner, "Invalid signature");
        
        // VULNERABILITY: Same signature can be used multiple times!
        balances[recipient] += amount;
    }
    
    function toEthSignedMessageHash(bytes32 hash) internal pure returns (bytes32) {
        return keccak256(abi.encodePacked("\x19Ethereum Signed Message:\n32", hash));
    }
    
    function recoverSigner(bytes32 hash, uint8 v, bytes32 r, bytes32 s) internal pure returns (address) {
        return ecrecover(hash, v, r, s);
    }
}

// SECURE: Nonce-based signature (each signature can only be used once)
contract SecureSignature {
    mapping(address => uint256) public balances;
    mapping(bytes32 => bool) public usedSignatures;
    address public owner;
    uint256 public nonce;
    
    constructor() {
        owner = msg.sender;
    }
    
    function verifyTransfer(
        address recipient,
        uint256 amount,
        uint256 _nonce,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) public {
        // Include nonce in the message hash
        bytes32 messageHash = keccak256(abi.encodePacked(recipient, amount, _nonce));
        bytes32 ethSignedHash = toEthSignedMessageHash(messageHash);
        address signer = recoverSigner(ethSignedHash, v, r, s);
        
        require(signer == owner, "Invalid signature");
        require(_nonce == nonce, "Invalid nonce");
        
        // Mark signature as used
        usedSignatures[ethSignedHash] = true;
        nonce++;
        
        balances[recipient] += amount;
    }
    
    function toEthSignedMessageHash(bytes32 hash) internal pure returns (bytes32) {
        return keccak256(abi.encodePacked("\x19Ethereum Signed Message:\n32", hash));
    }
    
    function recoverSigner(bytes32 hash, uint8 v, bytes32 r, bytes32 s) internal pure returns (address) {
        return ecrecover(hash, v, r, s);
    }
}
```

## EIP-712: Typed Structured Data

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// EIP-712 provides a standard for signing structured data
// This prevents phishing by showing users exactly what they're signing

contract EIP712Example {
    bytes32 public constant DOMAIN_TYPEHASH = keccak256(
        "EIP712Domain(string name,string version,uint256 chainId,address verifyingContract)"
    );
    
    bytes32 public constant TRANSFER_TYPEHASH = keccak256(
        "Transfer(address recipient,uint256 amount,uint256 nonce)"
    );
    
    bytes32 public DOMAIN_SEPARATOR;
    mapping(address => uint256) public nonces;
    
    constructor() {
        DOMAIN_SEPARATOR = keccak256(
            abi.encode(
                DOMAIN_TYPEHASH,
                keccak256(bytes("MyToken")),
                keccak256(bytes("1")),
                block.chainid,
                address(this)
            )
        );
    }
    
    function verifyTransfer(
        address recipient,
        uint256 amount,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) public {
        bytes32 structHash = keccak256(
            abi.encode(TRANSFER_TYPEHASH, recipient, amount, nonces[msg.sender])
        );
        
        bytes32 digest = keccak256(
            abi.encodePacked("\x19\x01", DOMAIN_SEPARATOR, structHash)
        );
        
        address signer = ecrecover(digest, v, r, s);
        require(signer == msg.sender, "Invalid signature");
        
        nonces[msg.sender]++;
        // Process transfer...
    }
}
```

## Key Takeaways
- **Never Reuse Signatures:** Always include a nonce or unique identifier in signed messages.
- **EIP-712:** Use typed structured data signing to prevent phishing and improve UX.
- **Domain Separation:** Include chain ID and contract address to prevent cross-chain/cross-contract replay.
- **Expiry Times:** Add expiration timestamps to signatures to limit their validity window.
- **User Education:** Warn users about signing unknown messages in their wallets.
- **MetaTransactions:** Signatures enable gasless transactions (users sign, relayers pay gas).