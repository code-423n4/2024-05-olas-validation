
L-01 VoteWeighting::voteForNomineeWeightsBatch() can return wrong values during revert
```solidity
    function voteForNomineeWeightsBatch(
        bytes32[] memory accounts,
        uint256[] memory chainIds,
        uint256[] memory weights
    ) external {
        if (accounts.length != chainIds.length || accounts.length != weights.length) {
>           revert WrongArrayLength(accounts.length, weights.length);
        }
```
If the `chainIds.length` is the array that cause th eerror, it will not be shown during the revert, which will lead to an incorrect revert error being displayed.