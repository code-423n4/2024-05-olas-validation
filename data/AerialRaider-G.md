Mark the owner variable as immutable, because it is only set in the constructor and never changed afterward.

https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol#L163

### Original Variable Declarations:

address public owner;

### Optimized Variable Declarations:

address public immutable owner;

This can save gas because it allows these variables to be stored directly in the contract's bytecode instead of using a storage slot.
