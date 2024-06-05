 Since all votes will be calulated to the next week, in the construct function of VoteWeighting.sol, "timeSum = block.timestamp / WEEK * WEEK;" could be changed to "timeSum = (block.timestamp + WEEK) / WEEK * WEEK;", in order to reduce the number of cycles.

https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol#L214