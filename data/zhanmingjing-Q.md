https://github.com/code-423n4/2024-05-olas/blob/main/governance/contracts/VoteWeighting.sol#L492

In function voteForNomineeWeights(), if lockEnd equals to nextTime, vote will revert. If lockEnd equals to nextTime, it means vote power is valid in current week, vote in this week should be valid.

So, VoteWeighting.sol#L492 should better be:

if (nextTime > lockEnd) 
