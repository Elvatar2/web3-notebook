# Multisig Wallets for Smart Contract Security

## Simple Definition
A multisig (multi-signature) wallet requires multiple private keys to authorize a transaction. Instead of one person controlling all funds, a predefined number of signers (e.g., 3 out of 5) must approve before any action executes.

## The Best Analogy
Think of a multisig like a **bank vault requiring multiple keys**. The vault has 5 keyholes, but only 3 keys need to be turned simultaneously to open it. No single person can access the vault alone - it requires collaboration. This prevents any one individual from acting maliciously or making mistakes.

## Code Example: Simple Multisig Wallet

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MultisigWallet {
    address[] public owners;
    uint256 public required; // Number of required confirmations
    uint256 public transactionCount;
    
    struct Transaction {
        address to;
        uint256 value;
        bytes data;
        bool executed;
        uint256 confirmations;
    }
    
    mapping(uint256 => Transaction) public transactions;
    mapping(uint256 => mapping(address => bool)) public confirmations;
    
    event Deposit(address indexed sender, uint256 amount);
    event SubmitTransaction(address indexed owner, uint256 indexed txIndex);
    event ConfirmTransaction(address indexed owner, uint256 indexed txIndex);
    event ExecuteTransaction(address indexed owner, uint256 indexed txIndex);
    
    constructor(address[] memory _owners, uint256 _required) {
        require(_owners.length > 0, "Owners required");
        require(_required > 0 && _required <= _owners.length, "Invalid required");
        
        for (uint i = 0; i < _owners.length; i++) {
            address owner = _owners[i];
            require(owner != address(0), "Invalid owner");
            require(!_isOwner(owner), "Duplicate owner");
            owners.push(owner);
        }
        
        required = _required;
    }
    
    receive() external payable {
        emit Deposit(msg.sender, msg.value);
    }
    
    function _isOwner(address _owner) internal view returns (bool) {
        for (uint i = 0; i < owners.length; i++) {
            if (owners[i] == _owner) return true;
        }
        return false;
    }
    
    modifier onlyOwner() {
        require(_isOwner(msg.sender), "Not an owner");
        _;
    }
    
    modifier txExists(uint256 _txIndex) {
        require(_txIndex < transactionCount, "Transaction does not exist");
        _;
    }
    
    modifier notExecuted(uint256 _txIndex) {
        require(!transactions[_txIndex].executed, "Transaction already executed");
        _;
    }
    
    modifier notConfirmed(uint256 _txIndex) {
        require(!confirmations[_txIndex][msg.sender], "Transaction already confirmed");
        _;
    }
    
    function submitTransaction(address _to, uint256 _value, bytes memory _data) 
        public 
        onlyOwner 
    {
        uint256 txIndex = transactionCount;
        
        transactions[txIndex] = Transaction({
            to: _to,
            value: _value,
            data: _data,
            executed: false,
            confirmations: 0
        });
        
        transactionCount++;
        emit SubmitTransaction(msg.sender, txIndex);
    }
    
    function confirmTransaction(uint256 _txIndex) 
        public 
        onlyOwner 
        txExists(_txIndex) 
        notExecuted(_txIndex) 
        notConfirmed(_txIndex) 
    {
        Transaction storage txn = transactions[_txIndex];
        txn.confirmations++;
        confirmations[_txIndex][msg.sender] = true;
        
        emit ConfirmTransaction(msg.sender, _txIndex);
    }
    
    function executeTransaction(uint256 _txIndex) 
        public 
        onlyOwner 
        txExists(_txIndex) 
        notExecuted(_txIndex) 
    {
        Transaction storage txn = transactions[_txIndex];
        require(txn.confirmations >= required, "Cannot execute transaction");
        
        txn.executed = true;
        
        (bool success, ) = txn.to.call{value: txn.value}(txn.data);
        require(success, "Transaction failed");
        
        emit ExecuteTransaction(msg.sender, _txIndex);
    }
    
    function getTransactionCount() public view returns (uint256) {
        return transactionCount;
    }
}
```

## Real-World Multisig Solutions

```text
1. Gnosis Safe (now Safe):
   - Industry standard for multisig wallets
   - Used by most major DeFi protocols
   - Supports ERC-20, NFTs, and contract interactions
   - Beautiful UI for non-technical signers

2. Common Configurations:
   - 2-of-3: Small teams, personal funds
   - 3-of-5: Medium organizations
   - 5-of-9: Large protocols (e.g., MakerDAO)
   - 7-of-13: Critical infrastructure

3. Best Practices:
   - Distribute keys geographically (different countries)
   - Use hardware wallets (Ledger, Trezor) for each signer
   - Include at least one institutional signer (law firm, auditor)
   - Regular key rotation and backup procedures
```

## Key Takeaways
- **No Single Point of Failure:** Compromising one key doesn't compromise funds.
- **Governance Alignment:** Forces consensus before critical actions.
- **Gas Overhead:** Multisig transactions cost more gas due to multiple signatures.
- **Use Safe Protocol:** Don't build your own multisig - use audited solutions like Safe.
- **Combine with Timelock:** For maximum security, use Multisig + Timelock (next file).
- **Key Management:** The security of a multisig is only as strong as its weakest key holder.