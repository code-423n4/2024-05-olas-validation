# QA Report

## [NC-1] Misleading error thrown in `VoteWeighting#getNextAllowedVotingTimes()`

When there is a mismatch in the length of the `accounts`, `chainIds` and `voters` arrays args in the `getNextAllowedVotingTimes()` function, the function reverts with a `WrongArrayLength` error.

However, the error always reports the `accounts.length` and `chainIds.length`. This would be misleading if the mismatch is in the `voters.length` because the error will report equal lengths in that scenario.

**Recommendation:** introduce a new error that supports three array length args or use `Math.min(accounts.length, chainIds.length)` and `Math.max(chainIds.length, voters.length)` to report the mismatched lengths.