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
