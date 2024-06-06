## Report for Olas Audit - StakingNativeToken.sol Optimization

**Finding:**

The `receive` function in the StakingNativeToken.sol contract can be optimized for gas efficiency.

**Explanation:**

The original code [link](https://github.com/code-423n4/2024-05-olas/blob/e2a8bc31d2769bfb578a06cc64919ad369a82c08/registries/contracts/staking/StakingNativeToken.sol#L39-L49) performs unnecessary checks and updates when receiving zero-value deposits. Here's a breakdown:

1. **Redundant Check:** The line `msg.value >= 0` is redundant because Solidity guarantees `msg.value` to be non-negative.
2. **Wasted Storage Update:** If `msg.value` is zero, the contract updates `balance` and `availableRewards` with their existing values, essentially performing a no-op (useless operation) that consumes gas.

**Optimization:**

```
receive() external payable {
  if (msg.value > 0) {
    // ... existing logic to update balances and emit event ...
  }
}
```

This version only updates the state and emits the event if a positive amount (`msg.value > 0`) is received. This saves gas because modifying contract storage is expensive in Solidity.

**Recommendation:**

We recommend implementing the optimized version of the `receive` function to improve gas efficiency for the Olas platform.

## Code4rena Submission Report

### Introduction

This report evaluates the gas efficiency of using an `isRemoved` flag versus an additional array to manage nominee removals in the `VoteWeighting` contract. The findings suggest that utilizing an `isRemoved` flag is generally more gas-efficient due to lower storage costs and simpler update logic.

### Referenced Code Sections

- **Contract File**: [VoteWeighting.sol](https://github.com/code-423n4/2024-05-olas/blob/e2a8bc31d2769bfb578a06cc64919ad369a82c08/governance/contracts/VoteWeighting.sol#L215-L218)

### Gas Efficiency Analysis

#### Current Implementation

```solidity
// Push empty element to the zero-th index
setNominees.push(Nominee(0, 0));
// For symmetry, push empty element to the zero-th index in the removed Nominee set as well
setRemovedNominees.push(Nominee(0, 0));
```

#### Recommended Optimization

We recommend using an `isRemoved` flag within the `Nominee` struct to mark removed nominees. This approach reduces storage costs and simplifies nominee management.

#### Proposed Implementation

```solidity
struct Nominee {
    bytes32 account;
    uint256 chainId;
    bool isRemoved; // Flag to indicate if the nominee is removed
}
```

### Detailed Analysis

#### Storage Costs

- **Boolean Flag (`isRemoved`)**: Utilizes a single storage slot (256 bits) for the boolean flag, potentially less if tightly packed.
- **Additional Array**: Each element in the additional array requires a new storage slot, doubling the storage slots needed per nominee.

#### Computational Costs

- **Updating `isRemoved` Flag**: Cheaper due to the simplicity of boolean updates.
- **Array Operations**: More gas-intensive due to dynamic storage operations.

### Conclusion

Using an `isRemoved` flag is more gas-efficient, reducing both storage and computational costs. This approach minimizes complexity and optimizes gas usage for nominee management.

By implementing the recommended changes, the `VoteWeighting` contract can achieve better performance and lower transaction costs.

### Links to Code Changes

- [Original Code - Line 215-218](https://github.com/code-423n4/2024-05-olas/blob/e2a8bc31d2769bfb578a06cc64919ad369a82c08/governance/contracts/VoteWeighting.sol#L215-L218)

