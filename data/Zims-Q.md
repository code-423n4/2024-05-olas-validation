[QA-1]

## Ownership Transfer Mechanism

**Severity: Low (Quality Assurance)**

### Description:

The current implementation of the `changeOwner` function allows for direct transfer of ownership by the current owner. This process can potentially lead to accidental or malicious changes in contract ownership without proper confirmation from the new owner.

### Example Code Snippet:

https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/governance/contracts/VoteWeighting.sol#L366-L372

### Recommendation:

Implement a two-step ownership transfer mechanism. This involves:

1. The current owner proposing a new owner.
2. The proposed new owner explicitly accepting the ownership transfer.

### Proposed New Implementation:

```solidity
// Address of the proposed new owner
address public proposedOwner;

// Function to propose a new owner
function proposeNewOwner(address newOwner) external {
    // Check for the contract ownership
    if (msg.sender != owner) {
        revert OwnerOnly(msg.sender, owner);
    }
    proposedOwner = newOwner;
}

// Function for the proposed new owner to accept the ownership
function acceptOwnership() external {
    // Check that the caller is the proposed new owner
    if (msg.sender != proposedOwner) {
        revert ProposedOwnerOnly(msg.sender, proposedOwner);
    }
    owner = proposedOwner;
    proposedOwner = address(0);
}
```

### Rationale:

A two-step ownership transfer process is a well-recognized best practice that enhances the security and reliability of the contract. It ensures that ownership transfers are intentional and confirmed by all parties involved, thereby reducing the risk of errors or unauthorized changes.
