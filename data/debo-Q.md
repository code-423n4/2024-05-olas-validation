# Low-1 - Block Timestamp can be manipulated by miners
# Locations:
```URL
https://github.com/code-423n4/2024-05-olas/blob/e2a8bc31d2769bfb578a06cc64919ad369a82c08/governance/contracts/VoteWeighting.sol#L214
https://github.com/code-423n4/2024-05-olas/blob/e2a8bc31d2769bfb578a06cc64919ad369a82c08/governance/contracts/VoteWeighting.sol#L228
https://github.com/code-423n4/2024-05-olas/blob/e2a8bc31d2769bfb578a06cc64919ad369a82c08/governance/contracts/VoteWeighting.sol#L243
https://github.com/code-423n4/2024-05-olas/blob/e2a8bc31d2769bfb578a06cc64919ad369a82c08/governance/contracts/VoteWeighting.sol#L268
https://github.com/code-423n4/2024-05-olas/blob/e2a8bc31d2769bfb578a06cc64919ad369a82c08/governance/contracts/VoteWeighting.sol#L283
https://github.com/code-423n4/2024-05-olas/blob/e2a8bc31d2769bfb578a06cc64919ad369a82c08/governance/contracts/VoteWeighting.sol#L309
https://github.com/code-423n4/2024-05-olas/blob/e2a8bc31d2769bfb578a06cc64919ad369a82c08/governance/contracts/VoteWeighting.sol#L489
https://github.com/code-423n4/2024-05-olas/blob/e2a8bc31d2769bfb578a06cc64919ad369a82c08/governance/contracts/VoteWeighting.sol#L503-L504
https://github.com/code-423n4/2024-05-olas/blob/e2a8bc31d2769bfb578a06cc64919ad369a82c08/governance/contracts/VoteWeighting.sol#L542
https://github.com/code-423n4/2024-05-olas/blob/e2a8bc31d2769bfb578a06cc64919ad369a82c08/governance/contracts/VoteWeighting.sol#L554
https://github.com/code-423n4/2024-05-olas/blob/e2a8bc31d2769bfb578a06cc64919ad369a82c08/governance/contracts/VoteWeighting.sol#L605
https://github.com/code-423n4/2024-05-olas/blob/e2a8bc31d2769bfb578a06cc64919ad369a82c08/governance/contracts/VoteWeighting.sol#L658
```
## Impact
Detailed description of the impact of this finding.
The use of block.timestamp in smart contracts is known to be potentially manipulable by miners, as they have some leeway in setting the timestamp for the blocks they mine. This can lead to a range of attacks, particularly in contracts that rely on time-based conditions for critical functions. In the case of the VoteWeighting contract, functions that determine voting power, lock expiration, and nominee removal could be influenced by manipulated timestamps, potentially leading to unfair voting outcomes or unauthorised actions.
## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.

Replacing block timestamp with block number:
```sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.25;

// ... (other code remains unchanged)

contract VoteWeighting {
    // ... (other code remains unchanged)

    // Assuming an average block time of 13 seconds for Ethereum
    uint256 public constant AVERAGE_BLOCK_TIME = 13;
    uint256 public constant BLOCKS_PER_WEEK = WEEK / AVERAGE_BLOCK_TIME;

    // ... (other code remains unchanged)

    function voteForNomineeWeights(bytes32 account, uint256 chainId, uint256 weight) public {
        // ... (other code remains unchanged)

        // Calculate the nextTime in blocks instead of timestamp
        uint256 currentBlock = block.number;
        uint256 nextTime = (currentBlock + BLOCKS_PER_WEEK) / BLOCKS_PER_WEEK * BLOCKS_PER_WEEK;

        // ... (rest of the function remains unchanged, replace all block.timestamp with nextTime)

        // Record last action block number instead of timestamp
        lastUserVote[msg.sender][nomineeHash] = currentBlock;

        // ... (rest of the function remains unchanged)
    }

    function removeNominee(bytes32 account, uint256 chainId) external {
        // ... (other code remains unchanged)

        // Calculate the nextTime in blocks instead of timestamp
        uint256 currentBlock = block.number;
        uint256 nextTime = (currentBlock + BLOCKS_PER_WEEK) / BLOCKS_PER_WEEK * BLOCKS_PER_WEEK;

        // ... (rest of the function remains unchanged, replace all block.timestamp with nextTime)

        // ... (rest of the function remains unchanged)
    }

    function revokeRemovedNomineeVotingPower(bytes32 account, uint256 chainId) external {
        // ... (other code remains unchanged)

        // Use block.number instead of block.timestamp
        if (oldSlope.end > block.number) {
            changesWeight[nomineeHash][oldSlope.end] -= oldSlope.slope;
            changesSum[oldSlope.end] -= oldSlope.slope;
        }

        // ... (rest of the function remains unchanged)
    }

    // ... (other code remains unchanged)
}
```
## Tools Used
I used manual review.
## Recommended Mitigation Steps
Replace block.timestamp with block.number: Consider using block.number in conjunction with an average block time to estimate time-based conditions, as block.number is less prone to manipulation.
Use of Oracles for Time Checks: Implement a time oracle that provides a secure and standardised reference for the current time, reducing reliance on potentially manipulable sources.