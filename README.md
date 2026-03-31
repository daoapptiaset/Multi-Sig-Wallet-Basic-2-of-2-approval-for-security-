# Multi-Sig-Wallet-Basic-2-of-2-approval-for-security-
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MultiSigWallet {
    address[2] public owners;
    uint256 public requiredApprovals = 2;
    mapping(uint256 => mapping(address => bool)) public approvals;

    struct Transaction {
        address to;
        uint256 value;
        bool executed;
    }

    Transaction[] public transactions;

    constructor(address owner1, address owner2) {
        owners[0] = owner1;
        owners[1] = owner2;
    }

    function submitTransaction(address _to, uint256 _value) public {
        require(msg.sender == owners[0] || msg.sender == owners[1], "Not owner");
        transactions.push(Transaction(_to, _value, false));
    }

    function approveTransaction(uint256 txIndex) public {
        require(msg.sender == owners[0] || msg.sender == owners[1], "Not owner");
        approvals[txIndex][msg.sender] = true;
    }

    function executeTransaction(uint256 txIndex) public {
        Transaction storage tx = transactions[txIndex];
        require(!tx.executed, "Already executed");
        require(approvals[txIndex][owners[0]] && approvals[txIndex][owners[1]], "Not enough approvals");

        tx.executed = true;
        payable(tx.to).transfer(tx.value);
    }

    receive() external payable {}
}
