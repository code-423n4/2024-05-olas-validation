
https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol#L223
 function _getSum() internal returns (uint256) {
   @>     // t is always > 0 as it is set in the constructor
        uint256 t = timeSum;


timeSum = block.timestamp / WEEK * WEEK; it can be 0.

